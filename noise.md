# Table of contents
- [Table of contents](#table-of-contents)
- [The Noise Protocol Framework](#the-noise-protocol-framework)
  - [Cryptographic primitives](#cryptographic-primitives)
  - [Handshake patterns](#handshake-patterns)
  - [Message tokens](#message-tokens)
    - [Keys tokens](#keys-tokens)
    - [DH calculation tokens](#dh-calculation-tokens)
  - [Noise\_XX](#noise_xx)
  - [Noise\_IK](#noise_ik)

<br>

# The Noise Protocol Framework
The **Noise Protocol Framework**, sometimes referred to as **Noise** or **Noise Framework**, is a public domain cryptographic framework for creating secure communication protocols based on **Diffie–Hellman key exchange**.<br>

Components of the **Noise**:
1. **Handshake patterns**.
2. **Cryptographic primitives**.

<br>

## Cryptographic primitives
Noise uses a set of well-established **cryptographic primitives**:
- For **key exchange**: `ECDH`;
- For **AEAD**: `AESGCM` or `ChaChaPoly` (`ChaCha20` + `Poly1305`);
- **Hash functions**: `SHA256` or `BLAKE2`;

<br>

`ChaCha20` and `Poly1305` are two **different** cryptographic algorithms that are usually paired together (aka `ChaCha20-Poly1305` or `ChaChaPoly`) to reach **AEAD**:
- `ChaCha20` is a **stream cipher**;
- `Poly1305` is a **MAC**;

<br>

`XChaCha20` is an **extension** of `ChaCha20` designed to solve a single, critical vulnerability: the risk of nonce reuse.<br>

The core difference: **nonce size**:
- size of **nonce** in `ChaCha20` is **64**-bit or **96**-bit;
- size of **nonce** in `XChaCha20` is **192**-bit;

<br>

## Handshake patterns
Noise defines several **handshake patterns**. These patterns dictate how two parties establish a secure connection.<br>

<br>

Names of Noise patterns consist of **two letters**:
- **first letter**: the status of the **initiator** (the party starting the connection);
- **second letter**: the status of the **responder** (the party answering the connection);

<br>

Meaning of the letters:
- `N` (**None**): the *static key* is **not** *known* (anonymous);
- `K` (**Known**): the *static key* is **known** (pre-shared out-of-band, e.g., via a config file);
- `X` (**Xmitted** / **Transmitted**): the *static key* is **transmitted securely during the handshake**;
- `I` (**Immediate**): the **initiator immediately sends** their *static key* **in** the **first message**;

<br>

Comparison of **popular Noise patterns**:
- `NN`
  - *Who is Authenticated?*
    - neither party;
  - *Key Exchange Dynamic*
    - only ephemeral (temporary) keys are exchanged;
  - *Best Use Case*
    - total anonymity;
    - vulnerable to **MitM attacks**, but great for quick, unauthenticated encryption;
- `IK`
  - *Who is Authenticated?*
    - both parties
  - *Key Exchange Dynamic*
    - **initiator knows** the **responder's key** *ahead of time* and **transmits own key immediately**;
  - *Best Use Case*
    - 0-RTT (Zero Round-Trip Time) data encryption;
    - this is exactly how **WireGuard** works;
- `NK`
  - *Who is Authenticated?*
    - responder only
  - *Key Exchange Dynamic*
    - **initiator knows** the **responder's key** *ahead of time* but remains anonymous;
  - *Best Use Case*
    - Useful for a client securely connecting to a public server without identifying itself;
- `XX`
  - *Who is Authenticated?*
    - both parties
  - *Key Exchange Dynamic*
    - both parties hide and transmit their static keys during the handshake;
  - *Best Use Case*
    - mutual authentication with maximum privacy
    - active eavesdroppers cannot see the identities (public keys) of either party;
- `XK`
  - *Who is Authenticated?*
    - both parties
  - *Key Exchange Dynamic*
    - **initiator knows** the **responder's key**, and transmits their own key later in the handshake;
  - *Best Use Case*
    - protects the initiator's identity from passive network observers;


<br>

## Message tokens
Each **message token** describes **public key** or **DH operation**.<br>

All Noise patterns in terms of **message tokens** are [**here**](https://noiseexplorer.com/patterns/).<br>

<br>

### Keys tokens
- `e` (**ephemeral**): the **sender** generates **new ephemeral DH key pair** and sends the **public key**;
- `s` (**static**): the **sender** sends **static public key** (often encrypted);
  - if a secure channel has already been established by previous tokens, this key is automatically encrypted;

<br>

### DH calculation tokens
DH tokens always use **two letters**:
- the **first** letter represents the **type** of **local key**;
- the **second** letter represents the **type** of **remote key**;


- `ee`: **session symmetric key** is calculated by DH with the *local* **ephemeral** key and the *remote* **ephemeral** key;
  - provides basic encryption but **no** identity authentication;
- `es`: **session symmetric key** is calculated by DH with the *local* **ephemeral** key and the *remote* **static** key;
- `se`: **session symmetric key** is calculated by DH with the *local* **static** key and the *remote* **ephemeral** key;
- `ss`: **session symmetric key** is calculated by DH with the *local* **static** key and the *remote* **static** key;
  - provides mutual identity binding;
- `...`: everything **above** the `...` happens **before** the network connection starts;
  - `<- s` above `...` means the **responder** sends its **static public key** `s` to **initiator** over **independent channel** (**out-of-band public key**);
    - in other words, **initiator knows responder's static public key** `s`;

<br>

## Noise_XX
- ``
```bash
  -> e
  <- e, ee, s, es
  -> s, se
```

<br>

- *initiator*: `-> e`
  - `e`: *initiator* generates **ephemeral key pair** and sends **ephemeral public key**;
- *responder*: `<- e, ee, s, es`
  - `e`: *responder* sends its **ephemeral public key**;
  - `ee`: *responder* calculates **symmetric key** using its own **ephemeral key** and **ephemeral public key** of *initiator*;
  - `s`: *responder* sends **encrypted static key**;
  - `es`: *responder* calculates **symmetric key** using its own **ephemeral key** and **static public key** of *responder*;
- *initiator*: `-> s, se`
  - `s`: *initiator* sends **encrypted static key**;
  - `se`: *initiator* calculates **symmetric key** using its own **staic key** and **ephemeral public key** of *responder*;

<br>

## Noise_IK
```bash
  <- s
  ...
  -> e, es, s, ss
  <- e, ee, se
```

<br>

- *responder*: `<- s` above `...`
  - `s`: *responder* sends its **static public key** over independent channel;
- *initiator*: `-> e, es, s, ss`
  - `e`: *initiator* generates **ephemeral key pair** and sends **ephemeral public key**;
  - `es`: *initiator* calculates **symmetric key** using its own **ephemeral key** and **static public key** of *responder*;
  - `s`: *initiator* sends **encrypted static key**
  - `ss`: *initiator* calculates **symmetric key** using its own **staic key** and **staic key** of responder;
- *responder*: `<- e, ee, se`
  - `e`: *responder* generates **ephemeral key pair** and sends **ephemeral public key**;
  - `ee`: *responder* calculates **symmetric key** using its own **ephemeral key** and **ephemeral public key** of *initiator*;
  - `se`: *responder* calculates **symmetric key** using its own **staic key** and **ephemeral public key** of *initiator*;

<br>
