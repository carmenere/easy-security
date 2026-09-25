# Table of contents
- [Table of contents](#table-of-contents)
- [TLS](#tls)
  - [TLS handshake](#tls-handshake)
  - [Type of messages in TLS handshake](#type-of-messages-in-tls-handshake)
- [Types of DH in TLS](#types-of-dh-in-tls)
- [TLS extensions](#tls-extensions)
- [Проверка статуса сертификата](#проверка-статуса-сертификата)
  - [OCSP](#ocsp)
  - [OCSP Stapling](#ocsp-stapling)
  - [OCSP Must-staple](#ocsp-must-staple)
  - [CRL](#crl)
  - [OCSP](#ocsp-1)
- [HPKP](#hpkp)
- [HSTS](#hsts)

<br>

# TLS
## TLS handshake
|Client|Message|Server|
|:-----|:------|:-----|
|-->>|`ClientHello`||
||`ServerKeyExchange`|<<--|
||`CertificateRequest`|<<--|
||`ServerHelloDone`|<<--|
|-->>|`Certificate`||
|-->>|`ClientKeyExchange`||
|-->>|`CertificateVerify`||
|-->>|`ChangeCipherSpec`||
|-->>|`Finished`||
||`ChangeCipherSpec`|<<--|
||`Finished`|<<--|
|-->>|`Encrypted request`||
||`Encrypted response`|<<--|

<br>

## Type of messages in TLS handshake
- `ClientHello`
  - contains:
    - **version** of protocol
    - **client random**;
    - **session ID**;
    - supported **cipher suites**;
    - **compression** algorithm;
    - **TLS extensions**;
- `ServerHello`
    - chosen version of protocol
    - **server random**;
    - **session ID**;
    - chosen **cipher suite**;
    - chosen **compression** algorithm;
    - **TLS extensions**;
- `Certificate`
  - certificate of **server**;
- `ServerKeyExchange`
  - contains **parameters** of server side to obtain **session symmetric key** for encription;
  - **all parameters** are **signed** by **private key of server**;
  - parameters depend on chosen **key exchange** algorithm;
  - for **EDH**:
    - server sends `p`, `g` and its **public DH key**;
  - for **ECDH**:
    - chosen **elliptic curve**;
    - **public DH key**;
- `CertificateRequest`
  - server can request client certificate if mutual authentication is used;
- `ServerHelloDone`
  - menas than server has sent all its data;
- `Certificate`
  - certificate of **client** if it was requested by server in `CertificateRequest` message;
- `ClientKeyExchange`
  - contains **parameters** of client side to obtain **session symmetric key** for encription;
  - parameters depend on chosen **key exchange** algorithm;
  - for **RSA**:
    - client generates **48** bytes **session symmetric key**;
    - encrypts it with public key of server;
    - this approach is **not compatible** with **PFS**;
      - if an attacker **compromises** the server's **private key**, he will be able to decrypt all previously recorded sessions;
  - for **DH**:
    - client sends its **public DH key**;
- `CertificateVerify`
- `ChangeCipherSpec`
  - the **client** sends a **ChangeCipherSpec** message to notify the **server** that **all subsequent messages will be encrypted**;
- `Finished`
  - the **first** encrypted message in session;
  - it contains the **hash** value over all exchanged handshake messages;
- `ChangeCipherSpec`
  - similarly, the **server** sends a **ChangeCipherSpec** message to notify the **client** that **all subsequent messages will be encrypted**;
- `Finished`
  - similarly, it contains the **hash** value over all exchanged handshake messages;

<br>

# Types of DH in TLS
В протоколе **TLS** используется **3 модификации алгоритма DH**:
- **Anonymous DH** (**ADH**): использование алгоритма DH без аутентификации публичных ключей;
- **Fixed DH**: используется постоянный закрытый ключ a
  - параметры сервера (p,g,A) заранее подписываются закрытым ключом сервера;
  - это означает, что параметры фиксированы от сессии к сессии, а сервер всегда использует один и тот же закрытый ключ a.
- **Ephemeral DH** (**DH**): для каждой **новой** сессии генерируется **новый** сессионный ключ `a`;
  - для аутентификации открытых параметров DH используется **протокол подписи** (**DSA** или **ECDSA**);
  - для проверки подписи используется открытый ключ сертификата;

<br>

# TLS extensions
- **ALPN** (*Application-Layer Protocol Negotiation*)
  - allows the client and server to negotiate application protocol (**HTTP/1.1**, **HTTP/2** или **HTTP/3**) during handshake;

- **Certificate Status Request** (aka **OCSP Stapling**)
  - allows the client to request that the server provide a time-stamped OCSP response during the TLS handshake;
  - this eliminates the need for the client to contact the OCSP responder directly, improving privacy and performance.

- **Supported Groups** (formerly called **Elliptic Curves**)
  - indicates which elliptic curve groups the client supports for key exchange;
  - the order of groups in this extension indicates client preference;
    - by default the `x25519` is **preferred** for its security and **performance**;
  - modern **TLS 1.3** uses **ECDHE** (Elliptic Curve Diffie-Hellman Ephemeral) for **PFS**;
  - common groups include:
    - `x25519` (`Curve25519`)
    - `secp256r1` (`P-256`)
    - `secp384r1` (`P-384`)
    - `secp521r1` (`P-521`)

- **ECH** (*Encrypted Client Hello*)
  - hides the contents of the `ClientHello` — including **SNI** and **ALPN**;
  - **before ECH**, there was **Encrypted SNI** (**ESNI**), but **ESNI** is **obsoleted** now;
  - the key to encrypt the `ClientHello` is distributed by by using a separate channel - DNS: the *ECH configuration* is published in **DNS record**;
    - when a client wants to connect to **example.com**, it first queries DNS, which contains the *ECH configuration*;
  - the `ClientHello` message part is split into two separate messages: an **inner part** and an **outer part**;
  - the **outer** part contains the **non-sensitive information** such as which ciphers to use and the TLS version, it also includes an **outer SNI**;
  - the **inner** part is **encrypted** and contains an **inner SNI**;
  - there are 2 approaches to **protect DNS**:
    - use **ECH** with **DNSSEC**
    - **ECH** + **DoH** (*DNS over HTTPS*)

- **Session tickets**
  - enables TLS **session resumption**;
  - **server** encrypts the **session state** by its private key and sends it to the client as a **ticket**;
  - **ticket** contains:
    - *cipher suites*;
    - *session symmetric key*;
  - when resuming, the **client** sends its **ticket** through **Session tickets** TLS extension inside `ClientHello`;
  - the **server** decrypts it to restore the session;
  - but it **breaks PFS**:
    - if an attacker **compromises** the server he gets the key by which all tickets were encrypted;
    - the key for encrypting **Session tickets** must be **rotated**, for example every 12 hours;

- **SNI** (*Server Name Indication*)
  - contains unencrypted target **hostname**;
  - when server maintains more that one domain on one IP it does not know which certificate to use and **SNI** provides this info for server;

- **Supported versions**
  - allows client to indicate which TLS versions they support

- **TLS False Start**
  - allows *client* to send *application data* to *server* before completing the TLS handshake, i.e. *client* **does not wait** `ChangeCipherSpec` and `Finished`, instead it sends *application data* **immediately after** `Finished`;
  - this approach reduces the round trip time (RTT) for sending data;

<br>

# Проверка статуса сертификата
Существует 2 основных протокола для проверки статуса сертификата
- CRL (Certificate Revocation List).
- OCSP (Online Certificate Status Protocol).

У каждого CA имеется список CRL, представляющий собой набор серийных номеров сертификатов. 
Недостатки CRL:
- CRL обновляется не очень быстро.
- CRL имеет большой размер.

<br>

## OCSP
OCSP решает некоторые проблемы CRL, в частности позволяет проверить статус одного конкретного сертификата. 

Недостатки OCSP: 
- OCSP респондер должен справляться с очень большой нагрузкой.
- OCSP респондер должен гарантировать свою доступность в любое время.
- Проверка сертификата является блокирующей операцией, поэтому время отклика OCSP респондера может влиять на время TLS Handshake.
- Приватность: центр сертификации получает информацию о том, какие сайты посещает пользователь.

Можно легко создать ситуацию, когда OCSP респондер станет недоступным для пользователя (например, добавить в hosts запись, которая резолвит OCSP респондер в 127.0.0.1). Поэтому в браузерах используется проверка статуса сертификата с частичным сбоем. Т.е. браузер попытается проверить статус сертификата, но если ответ не придёт за некоторый промежуток времени, то браузер считает, что сертификат не отозван. А некоторые бразуеры вовсе не используют OCSP, а вместо OCSP используют собственные механизмы, например, в  chrome – CRLsets, в FF – OneCRL.

<br>

## OCSP Stapling
Для снижения времени отклика от OCSP респондера и оптимизации TLS Handshake был разработан механизм OCSP Stapling, который заключается в том, что сервер отправляет свой сертификат вместе с ответом OCSP responder’а в виде расширения TLS.
Ответ OCSP респондера должен быть подписан CA.

<br>

## OCSP Must-staple
OCSP Must-Staple – это флаг в сертификате, который указывает браузеру, что сертификат должен поставляться вместе с OCSP Stapling либо данный сертификат будет отвергнут.

Установить флаг легко, для этого надо попросить CA установить данный флаг.

В случае компрометации сертификата с установленным флагом OCSP Must-Staple, злоумышленник не сможет его использовать, т.к. ему придется использоваться OCSP Staple, а если он его включит, то OCSP респондер скажет, что сертификат отозван.


<br>

## CRL
CRL stands for Certificate Revocation List; it provides the means to check the revocation status of a certificate installed on a website or used to digitally sign a document. CRLs are binary files that contain the serial numbers of revoked certificates and in some cases a revocation reason. Each time a revocation check is performed, the client applications needs the CRL from the Issuing CA.  In come cases this may be cached from recent checks, but generally the CRL must be downloaded in full and searched. Over time, the CRLs grow as the number of certificates are revoked and this results in large CRLs and increased latency during the TLS handshake.

A CRL is an object which contains the list of serial numbers of certificates which have been revoked by a given CA. It is a signed object; the CRL issuer is usually the CA itself (with the same key) but this power can be delegated.

A CRL is a large, periodically updated file listing revoked serial numbers. CRLs are slow to fetch and often stale.

Historically, clients downloaded Certificate Revocation Lists (CRLs).

When a browser connects over HTTPS, it must trust the certificate chain. That trust is not permanent. Certificates get compromised, keys leak, and domains change hands. Revocation tells clients to reject certificates that should no longer be trusted, even if the expiry date has not passed.

<br>

## OCSP
OCSP (Online Certificate Status Protocol, defined in RFC 6960) replaced that model with a lightweight request: the client sends the certificate serial number to an OCSP responder and receives a signed good, revoked, or unknown status.

In practice, plain OCSP creates problems. Each browser may contact the CA responder directly. That adds latency to the TLS handshake.
It also leaks which sites users visit to the CA.
If the responder is slow or down, browsers face a hard choice: fail open (accept the cert) or fail closed (block the site). Neither outcome is ideal for production traffic.

OCSP stapling (formally TLS Certificate Status Request, RFC 6066) shifts that work to the server. The web server periodically fetches a signed OCSP response from the CA and staples it to the TLS handshake. The browser validates the staple against the certificate chain it already received. No extra round trip to the CA. No browsing telemetry sent upstream.

An OCSP response is what an OCSP responder returns when it receives a request about the revocation status of a certificate.

OCSP stapling is a way for a SSL server to obtain OCSP responses for his own certificate, and provide them to the client, under the assumption that the client may need them. This makes the whole process more efficient: the client does not have to open extra connections to get the OCSP responses itself, and the same OCSP response can be sent by the server to all clients within a given time frame. One way to see it is that the SSL server acts as a Web proxy for the purpose of downloading OCSP responses.

OCSP or Online Certificate Status Protocol addresses some of the performance and scalability issues inherent to CRLs. Instead of having to download a full revocation list each time, the OCSP server can be queried like a database for a specific certificate entry. The OCSP response is signed by the CA and contains a status for the certificate.

<br>

The Certificate Status Request extension, also known as OCSP Stapling, allows the client to request that the server provide a time-stamped OCSP response during the TLS handshake. This eliminates the need for the client to contact the OCSP responder directly, improving privacy and performance. The server "staples" the OCSP response to the certificate chain, reducing latency and preventing OCSP responder overload. OCSP Stapling is particularly important for mobile and high-traffic websites. Defined in RFC 6066.

OCSP stapling is designed to reduce the cost of an OCSP validation

<br>

# HPKP
HTTP Public Key Pinning (HPKP)

<br>

# HSTS
HTTP Strict Transport Security (HSTS)