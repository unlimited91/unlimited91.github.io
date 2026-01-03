---
layout: post
title: "Primer on Elliptic Curve Math"
date: 2026-01-03
---

I'm pretty sure many of you have used asymmetric cryptography in various ways. More often than not, it's a requirement for a project, and we think, "Yeah, there's this neat trick to ensure point-to-point encryption." The feature gets written, tested, and deployed. We're like, "Yeah, whatever," and move on to the next project. But I'm sure a lot of us have never bothered to understand what's happening under the hood, especially the math behind Elliptic Curve Cryptography. I understand this is a slippery slope, and I'm by no means an expert, but I've tried some tinkering :). Today this is going to be an extremely high-level overview.

## Now let's begin

Imagine a scenario where there are two backend systems: Backend A and Backend B. Backend B exposes an API called `getCardDetails` which is supposed to return sensitive card details. Backend A is going to call Backend B to fetch this information. Now we want to ensure that the data in transit is secure and no one can eavesdrop on it.

Let's look at a diagram to understand this better.

![Backend A and Backend B setup diagram](/assets/img/setup.png)

![Backend A and Backend B api flow diagram](/assets/img/apicall.png)

I think the diagram is pretty self explanatory and for someone who has never worked with cryptography this would seem like magic! What the hell is going on here?

Well let's dive into the math of it.

### Elliptic Curve Definition

The elliptic curve equation is:

$$E: y^2 = x^3 - 3x + b \pmod{p}$$

**Key properties:**

- Group of points on the curve
- Base point $G$ of prime order $n$
- Cofactor $h = 1$

This makes the group algebraically clean and safe.

### Keys as Scalars and Points

A **private key** is a random scalar:

$$d \in [1, n-1]$$ 

A **public key** is scalar multiplication:

$$Q = [d]G$$

So Private Key = $d$ and Public Key = $Q$.

### Now the Key Exchange

Let:

### Backend A 
- Backend A generated an ephemeral keypair therefore
- Backend A's ephemeral private key = $e_A$
- Backend A's ephemeral public key = $Q_A = [e_A]G$

### Backend B
- Backend B generated an ephemeral keypair therefore
- Backend B's ephemeral private key = $e_B$
- Backend B's ephemeral public key = $Q_B = [e_B]G$

### Backend B's shared secret computation
K = (Private Key of Backend B) * (Public Key of Backend A)

$$K = [e_B]Q_A $$

which basically boils down to:

$$K = [e_B][e_A]G $$

### Backend A's shared secret computation
K = (Private Key of Backend A) * (Public Key of Backend B)

$$K = [e_A]Q_B $$

which basically boils down to:

$$K = [e_A][e_B]G $$

Because scalar multiplication is associative and commutative:

$$[e_B][e_A]G = [e_A][e_B]G$$

Both sides arrive at the same elliptic-curve point. F*** amazing!

Now of course ECDH gives a point, not a key.

Therefore, typically one would do the following:

Extract the x-coordinate

Feed it into a KDF:

$$ K = HKDF(𝑥,salt,info) $$

This produces cryptographically uniform key material.

For the astute ones let's write a quick python script and validate it on sagemath

Open Sagemath (https://sagecell.sagemath.org/) and just copy the following python script. I have the script pretty self explanatory.

```python
p  = 0xffffffff00000001000000000000000000000000ffffffffffffffffffffffff
a  = -3
b  = 0x5ac635d8aa3a93e7b3ebbd55769886bc651d06b0cc53b0f63bce3c3e27d2604b

Gx = 0x6b17d1f2e12c4247f8bce6e563a440f277037d812deb33a0f4a13945d898c296
Gy = 0x4fe342e2fe1a7f9b8ee7eb4a7c0f9e162bce33576b315ececbb6406837bf51f5

n  = 0xffffffff00000000ffffffffffffffffbce6faada7179e84f3b9cac2fc632551
h  = 1

Fp = GF(p)
E  = EllipticCurve(Fp, [a, b])
G  = E(Gx, Gy)
O = E(0)  # point at infinity in Sage. More about this later in a detailed post.

def validate_point(P):
    if P == O:
        return False, "infinity"
    if P not in E:
        return False, "not on curve"
    # subgroup / order check:
    if (n * P) != O:
        return False, "wrong subgroup (nP != O)"
    return True, "ok"

# --- Generate static keypairs ---
def gen_keypair():
    d = ZZ.random_element(1, n)  # private scalar in [1, n-1]
    Q = d * G                    # public point
    ok, why = validate_point(Q)
    if not ok:
        raise Exception("Generated invalid pubkey?! " + why)
    return d, Q

dA, Apub = gen_keypair()
dB, Bpub = gen_keypair()

print("\nStatic pub validation:")
print("Apub:", validate_point(Apub))
print("Bpub:", validate_point(Bpub))

Backend_A_Shared_Key = dA * Bpub   # (Private Key of Backend A) * (Public Key of Backend B)
Backend_B_Shared_Key = dB * Apub   # (Private Key of Backend B) * (Public Key of Backend A)

print("Backed A derived shared key", Backend_A_Shared_Key)
print("Backed B derived shared key", Backend_B_Shared_Key) # you'll see both of them are the same```
