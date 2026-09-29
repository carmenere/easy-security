# Table of contents
- [Table of contents](#table-of-contents)
- [ECC](#ecc)
- [Elliptic curves](#elliptic-curves)
- [Параметры эллиптической кривой](#параметры-эллиптической-кривой)
  - [Curve25519](#curve25519)
- [Curve25519-based cryptosystems](#curve25519-based-cryptosystems)

<br>

# ECC
**EC** – **E**lliptic **C**urve.<br>
**ECC** – **E**lliptic **C**urve **C**ryptography.<br>

<br>

**Эллиптическая криптография** — раздел криптографии, который изучает **асимметричные криптосистемы**, основанные на **эллиптических кривых над конечными полями**.<br>

<br>

**Key agreement** and **digital signatures** based on **EC**:
- **ECDH** - **EC** version of **DH** (*Diffie–Hellman*), i.e. **DH key exchange** based on **EC**;
- **ECDSA** - **EC** version of **DSA**, i.e. **DSA** (*Digital Signature Algorithm*) based on **EC**;
  - **EdDSA** (**Edwards-curve** version of **ECDSA**) - is an **ECDSA** based on **twisted Edwards curves**;

<br>

# Elliptic curves
There are **3 main algebraic forms** of **elliptic curve**:
- **short Weierstrass** form:
  - $`y^{2} = x^{3} + ax + b`$
- **Montgomery** form:
  - $`B \cdot y^{2} = x^{3} + A \cdot x^{2} + x`$
- **twisted Edwards** curves:
  - $`a \cdot x^{2} + y^{2} = 1 + d \cdot x^{2} \cdot y^{2}`$

<br>

Each **form** to represents a **class** of elliptic curves: **Weierstrass curves**, **Montgomery curves** and  **Edwards curves**.<br>

<br>

Relationships:
- **all** *elliptic curves* can be written in *Weierstrass form*;
- **not all** *Weierstrass curves* curves have a *Montgomery form* or *twisted Edwards form*;
- **every** *Montgomery curve* has a *twisted Edwards form*, and **vice versa**;

<br>

# Параметры эллиптической кривой
Для описания эллиптической кривой в стандарте NIST используется набор из **6 параметров**: $`D=(p,a,b,G,n,h)`$, где
- $`p`$ — **модуль эллиптической кривой** (**prime**) - **простое число**, которое задает **размер конечного поля**, в котором выполняются все вычисления точек кривой;
  - все координаты точек $`(x, y)`$ и коэффициенты $`a, b`$ уравнения кривой берутся по остатку от деления на **модуль эллиптической кривой** $`p`$;
  - **модуль эллиптической кривой** задаёт размер конечного поля $`F_{p}`$, в котором выполняются все вычисления (остатки от деления на $`p`$);
  - **пордяок эллиптической кривой** - **общее количество точек на кривой**, **образующих группу над конечным полем**;
  - чем больше число $`p`$, тем сложнее взломать систему методом перебора
- $`a, b`$ — коэффициенты уравнения эллиптической кривой;
- $`G`$ — **базовая точка** эллиптической кривой (aka **генератор**);
- $`n`$ — **порядок базовой точки** $`G`$;
  - т.к. *генератор* $`G`$ порождает **циклическую подгруппу**, порядок которой **равен** порядоку базовой точки;
  - т.о. **порядок базовой точки** $`G`$ - это **количество всех точек на кривой**, **порожденных** *базовой точкой* $`G`$;
- $`h`$ — **кофактор** - отношение **порядка эллиптической кривой** к **порядку базовой точки** $`n`$;
  - данное число должно быть **как можно меньше**;

<br>

## Curve25519
In 2005, **Curve25519** was first released by *Daniel J. Bernstein*. It is **Montgomery curve** with the **prime** $`p = 2^{255}-19`$.<br>
Older curves are **P-256**, **P-384** and **P-521**.<br>

<br>

Curves **Curve25519** and **Curve448** are included in **NIST Special Publication 800-186** as an **approved Montgomery curve for U.S. government use**.<br>
**Curve25519** is generally considered **safer** and **easier** to implement correctly than **NIST P-256**.<br>

<br>

**Curve25519** became extremely popular because it:
- it is **simpler** and **safer** to implement;
- it has **better resistance** against **side-channel attacks**;
- it **outperforms** traditional **NIST P-256**;

<br>

# Curve25519-based cryptosystems
**Curve25519-based cryptosystems**:
- **X25519** is an **implementation** of **ECDH** over **Curve25519**;
- **Ed25519** is an **implementation** of **EdDSA** based on the *twisted Edwards form* form of **Curve25519**;

So:
- **X25519** for Diffie-Hellman key exchange;
- **Ed25519** for digital signatures;

<br>

- Typical **Ed25519** sizes include:
  - *public key*: **32** bytes;
  - *private key*: **32** bytes;
  - *signature*: **64** bytes;
- Typical **X25519** sizes include:
  - *public key*: **32** bytes;
  - *private key*: **32** bytes;
  - *shared secret*: **32** bytes;

<br>

**Prformance**:
- **X25519** key exchange **outperforms** traditional **ECDH over P-256**;
- **Ed25519** signing **outperforms** **ECDSA over P-256**;
