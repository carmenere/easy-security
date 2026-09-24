# Table of contents
- [Table of contents](#table-of-contents)
- [Convert from one format to another](#convert-from-one-format-to-another)
- [Formats](#formats)
  - [Encodings](#encodings)
  - [X.509](#x509)
  - [PKCS](#pkcs)
    - [Keys](#keys)
    - [Containers](#containers)
- [OpenSSH and PuTTY](#openssh-and-putty)

<br>

# Convert from one format to another
- from `.p12` to `.p12`, for example, to **set** or **drop** password:
  - `openssl pkcs12 -in foo.p12 -out temp.pem -legacy -nodes`
    - reads `foo.p12` and extracts **keys** and **certificates** from it;
    - `-legacy` must be provided if `.p12` was created by old versions of openssl: `RC2`/`3DES`/`SHA1/...`;
  - `openssl pkcs12 -export -in temp.pem -out foo`
    - creates new **PKCS#12** file `foo` from `temp.pem`;

<br>

# Formats
## Encodings
**Encoding formats** define how **ASN.1** structure is serialized into bytes.<br>

There are 2 main formats:
- **PEM**
  - **textual Base 64 format** for *keys* and *certificates*;
  - it usually contains **DER inside Base64**;
  - widely used in Linux/OpenSSL;
  - **extensions**:
    - `.pem`
    - `.crt`
    - `.cer`
    - `.key`
- **DER**
  - **binary format** for *keys* and *certificates*;
  - widely used in Windows/Java;
  - **extensions**:
    - `.der`
    - `.cer`

<br>

## X.509
The `X.509` defines an **ASN.1 schema** for certificate.<br>
The `X.509` can be encoded with **DER** or **PEM**.<br>


<br>

## PKCS
**PKCS family** stands for **Public-Key Cryptography Standards**.<br>

<br>

All **PKCS#N** are **not** encoding formats, they all define **ASN.1 schemas**.

### Keys
- **PKCS#1**
  - defines **RSA-specific** **ASN.1 schema** for keys;
  - **headers**:
    - `-----BEGIN RSA PRIVATE KEY-----`
    - `-----BEGIN RSA PUBLIC KEY-----`
- **PKCS#8**
  - defines an **ASN.1 schema** for storing a **private key** only;
  - **headers**:
    - `-----BEGIN PRIVATE KEY-----`
    - `-----BEGIN ENCRYPTED PRIVATE KEY-----`
- **PKCS#10**
  - defines an **ASN.1 schema** for **CSR** (Certificate Signing Request);
  - extensions:
    - `.csr`
    - `.req`

<br>

### Containers
- **PKCS#7**
  - defines an **ASN.1 schema** of container for *certificates* and *chanin of certificates*, but **without private keys**;
  - **extensions**:
    - `.p7b`
    - `.p7c`
- **PKCS#12**
  - defines an **ASN.1 schema** of container for *certificates*, *chanin of certificates* and *private keys*;
  - can be **protected by password**;
  - **encoding**: **binary**;
  - **extensions**:
    - `.pfx`
    - `.p12`

<br>

# OpenSSH and PuTTY
- **OpenSSH**
  - describes format for keys for *OpenSSH*;
  - files:
    - `id_rsa`
    - `id_rsa.pub`
    - `id_ed25519`
    - `id_ed25519.pub`
    - and so on;
- **PuTTY**
  - describes format for keys for *PuTTY*;
  - **extensions**:
    - `.ppk`