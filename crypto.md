# Table of contents
- [Table of contents](#table-of-contents)
- [ECC](#ecc)
- [Угрозы в отношении данных в процессе их передачи](#угрозы-в-отношении-данных-в-процессе-их-передачи)
- [Message Authentication](#message-authentication)
- [HMAC](#hmac)
- [MAC based on encription](#mac-based-on-encription)
- [Digital signature](#digital-signature)
- [Key exchange](#key-exchange)
  - [Perfect forward secrecy](#perfect-forward-secrecy)
  - [DH](#dh)
- [Режимы блочного шифрования](#режимы-блочного-шифрования)
- [ECB](#ecb)
- [CBC](#cbc)
- [PCBC](#pcbc)
- [CFB](#cfb)
  - [General CFB-s](#general-cfb-s)
  - [Full-block CFB](#full-block-cfb)
- [OFB](#ofb)
- [CTR](#ctr)
- [Authenticated encryption with Associated Data (AEAD)](#authenticated-encryption-with-associated-data-aead)

<br>

# ECC
**EC** – Elliptic Curve.<br>
**ECC** – Elliptic Curve Cryptography.<br>

<br>

# Угрозы в отношении данных в процессе их передачи
1. Несанкционированный перехват данных: состоит в том, что злоумышленник перехватывает чужие данные.
2. Несанкционированная модификация данных: состоит в том, что злоумышленник модифицирует перехваченные данные и стремится уверить получателя в том, что они посланы законным пользователем.
2.1. Злоумышленник изменяет данные, оставляя контрольную сумму неизменной.
2.2. Злоумышленник создает поддельное сообщение и снабжает его новым значением контрольной суммы.
3. Несанкционированная отправка данных: состоит в том, что злоумышленник инициирует отправку поддельных данных и стремится уверить получателя в том, что они посланы законным пользователем.

<br>

Для защиты от несанкционированного перехвата данных (**1.1**) используется шифрование.
Для защиты от несанкционированной модификации/отправки одного шифрования недостаточно.
Для защиты от несанкционированной модификации/отправки (**2.1**, **2.2** и **3**) используются **методы защиты целостности данных** и/или **аутентификация источника данных**.

<br>

# Message Authentication
Аутентификация источника данных
Аутентификация источника данных тесно связана с защитой целостности данных, однако, на самом деле это совершенно разные понятия.
установления подлинности сообщения

Защита целостности данных в криптографии тесно связана с понятием код контроля ошибок из теории связи. 
Код контроля ошибок или Error detection code – процедура обнаружения и исправления случайных искажений данных, которые возникли при передачи сообщения.

В криптографии защита целостности данных предполагает использование мер, позволяющих обнаруживать не столько случайные искажения информации, сколько целенаправленное изменение информации активным криптоаналитиком.

Принципы защиты целостности данных и методы распознавания ошибок совпадают:
- отправитель сообщения вычисляет и присоединяет к сообщению контрольное значение;
- получатель, получив сообщение, проверяет контрольное значение;

Механизмы защиты целостности данных бывают симметричными и асимметричными.

<br>

Виды контрольных значений (сумм):
- **Manipulation Detection Code** (**MDC**) – контрольная сумма, вычисленная с помощью необратимой функции преобразования;
  - MDC обеспечивает **только** защиту целостности сообщения;
  - позволяет защититься от угрозы **2.1**;

- **Message Authentication Code** (**MAC**) или **Имитовставка** – **контрольная сумма**, при вычислении которой используется **секретный ключ**;
  - MAC обеспечивает как защиту **целостности сообщения** так и **аутентификацию источника сообщения**;
  - позволяет защититься от угроз **2.1**, **2.2** и **3**;
  - симметричные методы защиты целостности данных тоже называют **MAC**;

<br>

Типы **симметричных методов** защиты целостности данных:
- **MAC**, использующие функции **хэширования с ключом**, также называются **HMAC** (**Hash MAC**);
- **MAC**, использующие алгоритмы **блочного шифрования**, например, **CBC-MAC**;

Типы **ассиметричных методов** защиты целостности данных:
- **Цифровая подпись**;

<br>

# HMAC
**Общая идея алгоритмов HMAC**: `MAC = H(Key || Message || Key)`
- `H` – алгоритм хэширования;
- `Key` – ключ, добавляемый к сообщению `Massage`;

Рекомендуется добавлять ключ Key к сообщению Message с обоих сторон: в качестве префикса и суффикса. Если сообщение не защищено ключом с обоех сторон, то хэш-функции, использующие структуру Меркла-Дамгарда (SHA1, MD5 и др.), уязвимы к атакам Length Extension (атака удлинением сообщения). Атака Length Extension позволяет злоумышленнику модифицировать сообщение даже без знания ключа.

HMAC: https://datatracker.ietf.org/doc/html/rfc2104

Blake 2 можно использовать вместо HMAC, т.к. имеется режим хэширования с ключем, при этом в отличие от HMAC работает гораздо быстрее, т.к. не используется 2 прохода.

<br>

# MAC based on encription
Алгоритм блочного шифрования в режиме сцепления блоков зашифрованного текста позволяет получить функцию хэширования с ключом, а функция хэширования, полученная таким образом, называется функция MAC.

Общая идея функции MAC:
Пусть EK(m) – алгоритм блочного шифрования сообщения m с ключом k. Для того, чтобы аутентифицировать сообщение M, оно разбивается на блоки определенного размера mi (i = 1,2, …, N). Пусть С0 = IV – вектор инициализации – случайная строка, размер которой равен блоку шифрования. 
Блоки mi последовательно преобразуются в блоки шифрованного текста Сi последующему алгоритму:
Сi = EK(mi + Ci-1), i = 1,2, …, N.

Пару (IV, Ci) можно применять в качестве MAC, который присоединяется к сообщению перед отправкой.

<br>

# Digital signature
При использовании ассиметричных криптосистем, один ключ является закрытым и им владеет только один человек, а второй ключ является открытым и любой может его получить. То, что зашифровано закрытым ключом можно расшифровать только открытым ключом и наоборот.

Зашифрованный закрытым ключом текст может использоваться для создания MDC, при этом считается, что создать MDC может только владелец закрытого ключа. Процесс расшифровки представляет собой процесс верификации MDC. Такое применение ассиметричной криптосистемы называется цифровая подпись. Цифровая подпись, помимо защиты целостности данных позволяет установить автора сообщения и позволяет предотвратить отказ от авторства (non repudiation), т.е. отрицание своей связи с посланным сообщением.

<br>

# Key exchange
Обмен ключами
Организация защищенного канала для обмена секретными ключами.
Результатом работы протокола обмена ключами является распределенный ключ, зависящий от случайных значений, поступающих от всех участников протокола.
Наибольшее распространение получил протокол Диффи-Хеллмана или Diffie-Hellman (DH).

<br>

## Perfect forward secrecy
Perfect Forward Secrecy (PFS) или Прямая совершенная секретность – свойство протокола согласования ключа (key exchange), которое гарантирует, что сессионные ключи, полученные при помощи набора ключей долговременного пользования, не будут скомпрометированы при компрометации одного из долговременных ключей.

<br>

## DH
Первообразный корень (**primitive root**) по модулю $`m`$ – целое число $`g`$, такое, что:
- $`g^{\phi(m)} \equiv 1 \pmod{m}`$;
- и
- $`g^{k} \not \equiv 1 \pmod{m} \,\, \forall k : 1 \le k \le \phi(m)`$;

<br>

Другими словами, первообразный корень – это **образующий элемент** группы по модулю $`m`$.<br>

Если $`g`$ первообразный корень, то для любого числа $`a`$, взаимно простого с $`m`$, найдется такое целое число $`k`$, что: $`g^{k} \equiv a \pmod{m}`$.<br>
Такое число$` k`$ называется индексом или **дискретным логарифмом** числа $`a`$ по основанию $`g`$ по модулю $`m`$.<br>

<br>

краткое описание алгоритма DH
При работе алгоритма DH:
- **Alice**:
  - генерирует случайное натуральное число $`Private_A`$ – закрытый ключ;
  - совместно с **Bob** устанавливает открытые параметры $`p`$ и $`g`$ (обычно значения $`p`$ и $`g`$ генерируются на одной стороне и передаются другой), где:
    - $`p`$ является случайным простым числом;
    - $`(p-1)/2`$ также должно быть случайным простым числом (для повышения безопасности);
    - $`g`$ является первообразным корнем по модулю $`p`$ (называется еще генератор, также является простым числом);
  - вычисляет открытый ключ Public_A:
  - $`Public_A = g^{Private_A} \pmod{m}`$
  - обменивается открытыми ключами с **Bob**;
  - вычисляет общий сеансовый секретный ключ $`K`$, используя открытый ключ $`Public_B`$ Bob и свой закрытый ключ $`Private_A`$
    - $`K = Public_B^{Private_A} \pmod{m}`$
- $`К`$ получается равным с обеих сторон;

<br>

![DH](/img/DH.png)

<br>

Для согласования общего секретного ключа используется алгоритм **DH** либо **ECDH** (Elliptic Curve Diffie-Hellman Ephemeral).<br>

<br>

В протоколе **TLS** используется **3 модификации алгоритма DH**:
- **Anonymous DH** (**ADH**): использование алгоритма DH без аутентификации публичных ключей;
- **Fixed DH**: используется постоянный закрытый ключ a
  - параметры сервера (p,g,A) заранее подписываются закрытым ключом сервера;
  - это означает, что параметры фиксированы от сессии к сессии, а сервер всегда использует один и тот же закрытый ключ a.

- **Ephemeral DH** (**DH**): для каждой **новой** сессии генерируется **новый** сессионный ключ `a`;
  - для аутентификации открытых параметров DH используется **протокол подписи** (**DSA** или **ECDSA**);
  - для проверки подписи используется открытый ключ сертификата;

<br>

# Режимы блочного шифрования
- **Only encryption**
  - **ECB**: Electronic Codebook;
  - **CBC**: Cipher Block Chaining;
  - **PCBC**: Propagating Cipher Clock Chaining;
  - **CFB**: Cipher Feedback;
  - **OFB**: Output Feedback;
  - **CTR**: Counter Mode;
- **Authenticated encryption**: *confidentiality* + *integrity*
  - **CCM**: Counter with CBC-MAC;
  - **GCM**: Galois/Counter Mode;

<br>

*XTS* vs. *GCM*:
- *XTS* is **optimized for storage**, while *GCM* is **optimized for network**;

<br>

*CBC* vs. *CFB*:
- in **CBC**, plaintext is XORed with encrypted previous ciphertext;
- in **CFB**, the previous ciphertext is encrypted to make a **keystream**, and that keystream is XORed with plaintext;
  - **CFB** is like a **stream cipher** whose keystream depends on the previous ciphertext;

<br>

Modes that **require padding** for last block:
- **ECB**
- **CBC**
- **PCBC**

Arbitrary-length modes, i.e. they do **not** require padding:
- **OFB**
- **CTR**

<br>

**Parallel encryption**:
- **CTR**
- **ECB**
- **GCM**
- **XTS**

<br>

**Parallel decryption**:
- **CBC**
- **CTR**
- **ECB**
- **GCM**
- **XTS**

<br>

**Сообщение** $`M`$ – **исходный открытый текст**.<br>
Перед шифрованием, cообщение $`M`$ разбивается на $`N`$ **блоков** $`m_{i}`$ ($`i = 1, 2, …, N`$). Размер каждого блока $`m_{i}`$ составляет $`n`$ бит и равен размеру входного блока функции шифрования $`E_{K}(m)`$. Если размер **последнего** блока **меньше** $`n`$, то он **дополняется** до нужного размера.<br>

$`E_{K}(m)`$ – функция шифрования, на вход которой подается **блок открытого текста** $`m_{i}`$ и **ключ шифрования** $`K`$.<br>

**Сообщение** $`С`$ – **зашифрованный текст**. Сообщение $`С`$ получается путем конкатенации из блоков $`C_{i}`$ зашифрованного текста.

$`\oplus`$ – операция **XOR**, сумма по модулю 2.<br>

**Notation**:
- `b` = **block size**, e.g. **128** bits for AES;
- `s` = **feedback size**, e.g. **8** bits for **CFB-8**, **128** bits for **CFB-128**;
- $`P_{i}`$ - block `i` of **plaintext** of `b` bit;
  - each block of **plaintext** has the **same size** - `b` bit;
  - if **last** block is **less then** `b` bits it must be **padded**;
- $`C_{i}`$ - block `i` of **ciphertext**;
- $`E_{K}`$ - **encryption function** that recieves key $`K`$ and block of **plaintext** $`P_{i}`$;
- $`D_{K}`$ - **decryption function** that recieves key $`K`$ and block of **ciphertext** $`C_{i}`$;
- $`IV`$ - **initial vector** / **nonce**, i.e. some random number that is passed to encryption function;
  - it is does **not** secret and can be passed with **ciphertext**;
- the letter `O` just stands for **Output**: $`O_{1}`$ means the **first keystream block**;
- $`\oplus`$ - **XOR**;
- $`MSB_{s}(X)`$ returns **leftmost** `s` bits of `X` value;

<br>

# ECB
![ECB](/img/ECB.png)

<br>

**Advantages**:
- **no IV**;
- **each block** is **independent**;
- both *encryption* and *decryption* **can** be **parallelized**;

<br>

**Disadvantages**:
- **vulnerable to frequency analysis**, because it **leaks** the **statistical properties** and **structural patterns** of the plaintext;

<br>

**Encrypting** steps:
- $`C{1} = E_{K}(P{1})`$
- $`C{2} = E_{K}(P{2})`$
- $`C{3} = E_{K}(P{3})`$
- ...
- $`C_{i} = E_{K}(P_{i})`$

<br>

# CBC
![CBC](/img/CBC.png)

<br>

**Advantages**:
- **resistant to frequency analysis**;
- each ciphertext block **depends on all** previous plaintext blocks;
- **hides** patterns if a **random**/**unpredictable IV** is used;
- *decryption* **can** be **parallelized**;

<br>

**Disadvantages**:
- *encryption* is **sequential** and **cannot** be **parallelized**;
- IV must be unpredictable/random for encryption;

<br>

**Encrypting** steps:
- $`C_{0} = IV`$
- $`C_{1} = E_{K}(P_{1} \oplus C_{0})`$
- $`C_{2} = E_{K}(P_{2} \oplus C_{1})`$
- $`C_{3} = E_{K}(P_{3} \oplus C_{2})`$
- ...
- $`C_{i} = E_{K}(P_{i} \oplus C_{i-1})`$

<br>

# PCBC
![PCBC](/img/PCBC.png)

<br>

**Advantages**:
- **resistant to frequency analysis**;
- each ciphertext block **depends on all** previous plaintext blocks;
- **hides** patterns if a **random**/**unpredictable IV** is used;
- *decryption* **can** be **parallelized**;

<br>

**Disadvantages**:
- *encryption* is **sequential** and **cannot** be **parallelized**;
- IV must be unpredictable/random for encryption;

<br>

**Encrypting** steps:
- $`C_{0} = IV`$
- $`P_{0} = 0`$
- $`C_{1} = E_{K}(P_{1} \oplus P_{0} \oplus C_{0})`$
- $`C_{2} = E_{K}(P_{2} \oplus P_{1} \oplus C_{1})`$
- $`C_{3} = E_{K}(P_{3} \oplus P_{2} \oplus C_{2})`$
- ...
- $`C_{i} = E_{K}(P_{i} \oplus P_{i-1} \oplus C_{i-1})`$

<br>

# CFB
![CFB](/img/CFB.png)

<br>

There are 2 mode of **CFB**:
- **general CFB-s**;
- **full-block CFB**;

<br>

**Advantages**:
- **resistant to frequency analysis**;
- turns a *block cipher* into a *stream cipher*;
- **no padding needed**, handles **arbitrary-length data**;
- for `s = b`, *decryption* **can** be **parallelized**;

<br>

**Disadvantages**:
- *encryption* is **sequential** and **cannot** be **parallelized**;
- for `s < b`, *decryption* is also **sequential** and thus **cannot** be **parallelized**;

<br>

## General CFB-s
**Encrypting** steps:
  - **step 1**:
    - $`R_{0} = IV`$
    - $`C_{1} = P_{1} \oplus MSB_{s}( E_{K}(IV) )`$
    - $`R_{1} = (R_{0} << s) \, | \, C_{1}`$
  - **step 2**:
    - $`C_{2} = P_{2} \oplus MSB_{s}( E_{K}(R_{1}) )`$
    - $`R_{2} = (R_{1} << s) \, | \, C_{2}`$
  - **step 3**:
    - $`C_{3} = P_{3} \oplus MSB_{s}( E_{K}(R_{2}) )`$
    - $`R_{2} = (R_{2} << s) \, | \, C_{3}`$
  - **step i**:
    - $`C_{i} = P_{i} \oplus MSB_{s}( E_{}(R_{i-1}) )`$
    - $`R_{i} = (R_{i-1} << s) \, | \, C_{i}`$

<br>

## Full-block CFB
**CFB-128** for **AES-128** means `s` = `b`, in such case $`R_i = C_i`$ and $`C_{0} = IV`$.<br>

<br>

**Encrypting** steps:
  - **step 1**:
    - $`C_{1} = P_{1} \oplus E_{K}(IV)`$
  - **step 2**:
    - $`C_{2} = P_{2} \oplus E_{K}(C_{1})`$
  - **step 3**:
    - $`C_{3} = P_{3} \oplus E_{K}(C_{1})`$
  - **step i**:
    - $`C_{i} = P_{i} \oplus E_{}(C_{i-1})`$

<br>

# OFB
![OFB](/img/OFB.png)

<br>

**Advantages**:
- **resistant to frequency analysis**;
- *stream cipher*;
- *encryption* and *decryption* functions are **identical**;
- **no padding needed**, handles **arbitrary-length data**;

<br>

**Disadvantages**:
- **nonce**/**counter** must be **unique**, **reuse is catastrophic**;
- **counter must not wrap**, thus counter management is critical; 
- *encryption* and *decryption* **cannot** be **parallelized**;

<br>

**Keystream** steps:
- $`O_{1} = E_{K}(IV)`$
- $`O_{2} = E_{K}(O_{1})`$
- $`O_{3} = E_{K}(O_{2})`$
- ...
- $`O_{i} = E_{K}(O_{i-1})`$

<br>

**Encrypting** steps:
- $`C_{1} = P_{1} \oplus O_{1}`$
- $`C_{2} = P_{2} \oplus O_{2}`$
- $`C_{3} = P_{3} \oplus O_{3}`$
- ...
- $`C_i = P_{i} \oplus O_{i}`$

<br>

# CTR
![CTR](/img/CTR.png)

<br>

**Advantages**:
- **resistant to frequency analysis**;
- *stream cipher*;
- *encryption* and *decryption* functions are **identical**;
- **no padding needed**, handles **arbitrary-length data**;
- both *encryption* and *decryption* **can** be **parallelized**;

<br>

**Disadvantages**:
- **nonce**/**counter** must be **unique**, **reuse is catastrophic**;
- **counter must not wrap**, thus counter management is critical; 

<br>

**Counters** steps:
- $`T_{1} = IV`$
- $`T_{2} = IV + 1`$
- $`T_{3} = IV + 2`$
- ...
- $`T_{i} = IV + i`$

<br>

**Keystream** steps:
- $`O_{1} = E_{K}(T_{1})`$
- $`O_{2} = E_{K}(T_{2})`$
- $`O_{3} = E_{K}(T_{3})`$
- ...
- $`O_{i} = E_{K}(T_{i})`$

<br>

**Encrypting** steps:
- $`C_{1} = P_{1} \oplus O_{1}`$
- $`C_{2} = P_{2} \oplus O_{2}`$
- $`C_{3} = P_{3} \oplus O_{3}`$
- ...
- $`C_{i} = P_{i} \oplus O_{i}`$

<br>

# Authenticated encryption with Associated Data (AEAD)
**AEAD** – шифрование, при котором одновременно обеспечиваются: 
- **конфиденциальность данных**;
- **защита целостности данных**; 
- **аутентификация источника данных**;

Authenticated encryption (AE) is any encryption scheme which simultaneously assures the data confidentiality (also known as privacy: the encrypted message is impossible to understand without the knowledge of a secret key[1]) and authenticity (in other words, it is unforgeable:[2] the encrypted message includes an authentication tag that the sender can calculate only while possessing the secret key[1]). Examples of encryption modes that provide AE are GCM and CCM.[1]

Many (but not all) AE schemes allow the message to contain "associated data" (AD) which is not made confidential, but is integrity protected (i.e., readable, but tamperevident). A typical example is the header of a network packet that contains its destination address.

Подходы при создании алгоритмов AEAD:
- Encrypt-then-MAC (**EtM**): открытый текст сначала шифруется, а затем аутентифицируется зашифрованный текст. Значение MAC не шифруется;
- Encrypt-and-MAC (**E&M**): открытый текст шифруется и аутентифицируется одновременно;
- MAC-then-Encrypt (**MtE**): открытый текст сначала аутентифицируется, затем шифруется вместе с MAC;

<br>

**Encrypt-then-MAC**:<br>
![EtM](/img/EtM.png)

<br>
<br>

**Encrypt-and-MAC**:<br>
![EaM](/img/EaM.png)

<br>
<br>

**MAC-then-Encrypt**:<br>
![MtE](/img/MtE.png)

<br>
