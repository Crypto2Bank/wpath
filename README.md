# wpath
🔐 Cryptographic key recovery protocol using homomorphic properties of secp256k1
# 🔐 W-Path Cryptographic Protocol

**Author:** Omar El Foudali  
**Email:** elfoudaliomar@yahoo.com  
**Organization:** cryptotobank  
**Created:** 2026  
**License:** MIT
wallet address: 0x1675A6b2897a672b9b42D912BedA4b86B583507A
## 📖 Overview

W-Path is a cryptographic key recovery protocol based on the homomorphic properties of elliptic curve cryptography. It enables secure backup and recovery of private keys for Bitcoin, Ethereum, and other secp256k1-based blockchains.

## 🧮 The Mathematical Proof

The core innovation is the **Homomorphic Bridge**:

### Homomorphic Property Explained

Let $G$ be the generator point of the secp256k1 elliptic curve, $k$ a private key, and $w$ a secret recovery factor (witness).

The protocol relies on the following property:

$$
(k \cdot w) \cdot G = (k \cdot G) \cdot w
$$

#### Step-by-step:

1. **Key Generation:**
	- Private key: $k$
	- Public key: $K = k \cdot G$
2. **Witness Generation:**
	- Random witness: $w$
	- Witness public: $W = w \cdot G$
3. **Backup:**
	- Backup key: $k' = k \cdot w$
	- Backup public: $K' = k' \cdot G = (k \cdot w) \cdot G$
	- Alternatively: $K' = (k \cdot G) \cdot w = K \cdot w$

This allows secure recovery: knowing $w$ and $k'$, one can recover $k$.

### Why does this work?

Elliptic curve scalar multiplication is associative:

$$
(a \cdot b) \cdot G = a \cdot (b \cdot G) = b \cdot (a \cdot G)
$$

So, $(k \cdot w) \cdot G = k \cdot (w \cdot G) = w \cdot (k \cdot G)$.

---

## 💻 Code Example (Python)

Below is a minimal example using the `ecdsa` library:

```python
from ecdsa import SECP256k1, SigningKey

# Generate private key k and witness w
k = 0x123456789abcdef123456789abcdef123456789abcdef123456789abcdef1234
w = 0xabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcd
curve = SECP256k1
G = curve.generator

# Compute public keys
K = k * G
W = w * G

# Backup key
k_prime = k * w
K_prime = k_prime * G

# Alternatively, K_prime = (k * G) * w
K_prime_alt = w * K

assert K_prime == K_prime_alt
```

---

## ✍️ Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes (improve docs, add code, etc.)
4. Submit a Pull Request referencing the relevant issue

---

## ❓ FAQ

**Q: What is the main benefit of W-Path?**
A: It allows secure, cryptographically sound key recovery for secp256k1-based blockchains.

**Q: What libraries can I use?**
A: Any library supporting secp256k1 and elliptic curve operations (e.g., `ecdsa`, `secp256k1`, `bitcoinlib`).

**Q: What if I lose my witness $w$?**
A: Without $w$, the original private key $k$ cannot be recovered from the backup.

**Q: Is this protocol compatible with Bitcoin and Ethereum?**
A: Yes, both use secp256k1.

---

## 🛠️ Troubleshooting

- Ensure you use the correct curve parameters (secp256k1)
- Always keep your witness $w$ secure and backed up
- If you encounter math errors, check for integer overflows or library-specific quirks

---

## 🤝 New Contributors

Welcome! Please read the code, try the example, and open issues or PRs for any questions or improvements.

---

## 📚 References

- [SEC 2: Recommended Elliptic Curve Domain Parameters](https://www.secg.org/sec2-v2.pdf)
- [ecdsa Python library](https://github.com/warner/python-ecdsa)

---

For questions, open an issue or contact the author.
