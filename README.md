# 🔐 W-Path Cryptographic Protocol

**Cryptographic key recovery protocol using homomorphic properties of secp256k1**

---

**Author:** Omar El Foudali  
**Email:** elfoudaliomar@yahoo.com  
**Organization:** cryptotobank  
**Created:** 2026  
**License:** MIT  
**Wallet Address:** `0x1675A6b2897a672b9b42D912BedA4b86B583507A`

---

## 📖 Overview

W-Path is a cryptographic key recovery protocol based on the homomorphic properties of elliptic curve cryptography. It enables secure backup and recovery of private keys for Bitcoin, Ethereum, and other secp256k1-based blockchains.

### Key Features

- 🔐 **Secure Backup** - Create encrypted backups of private keys
- 🔄 **Key Recovery** - Recover lost keys using witness factor
- ⛓️ **Multi-Chain** - Works with Bitcoin, Ethereum, and all secp256k1 chains
- 🧮 **Mathematically Proven** - Based on well-established elliptic curve properties

---

## 🧮 The Mathematical Proof

The core innovation is the **Homomorphic Bridge**, which leverages the algebraic structure of elliptic curve scalar multiplication.

### Core Property

Let:
- `G` = generator point of secp256k1 curve
- `n` = order of the curve (≈ 2²⁵⁶)
- `k` = private key (scalar, 0 < k < n)
- `w` = secret witness factor (scalar, 0 < w < n)
- `K` = public key = k·G

The protocol relies on this fundamental identity:

```
(k · w) · G ≡ (k · G) · w    (mod n)
```

### Step-by-Step Proof

**1. Associativity of Scalar Multiplication:**

For any scalars a, b and point P on the curve:
```
(a · b) · P = a · (b · P)
```

**2. Commutativity in the Exponent:**

```
(k · w) · G = (w · k) · G = k · (w · G) = w · (k · G)
```

**3. The Homomorphic Bridge:**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Backup Path:          Recovery Path:                      │
│   ────────────          ──────────────                      │
│                                                             │
│   k ──→ k' = k·w        k' ──→ k = k'·w⁻¹                   │
│   │                      │                                   │
│   ↓                      ↓                                   │
│   K' = k'·G            K' = (k·w)·G                         │
│        │                    │                                │
│        │    ════════════    │                                │
│        │    EQUIVALENT      │                                │
│        │    ════════════    │                                │
│        ↓                    ↓                                │
│   K' = (k·w)·G = (k·G)·w = K·w                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Key Insight:** The backup key `k' = k·w` produces the same public key whether computed as `(k·w)·G` or `(k·G)·w`.

### Modular Arithmetic Note

⚠️ **Important:** All scalar operations are performed modulo `n` (the curve order):

```
k' = (k · w) mod n
k = (k' · w⁻¹) mod n
```

Where `w⁻¹` is the modular multiplicative inverse of `w` modulo `n`.

---

## 💻 Code Examples

### Python Example

```python
"""
W-Path Key Backup and Recovery Example
Requires: pip install ecdsa
"""
from ecdsa import SECP256k1, SigningKey
from ecdsa.util import string_to_number

# secp256k1 curve parameters
curve = SECP256k1
n = curve.order  # Curve order
G = curve.generator

def mod_inverse(a: int, m: int) -> int:
    """Compute modular multiplicative inverse using extended Euclidean algorithm"""
    def extended_gcd(a, b):
        if a == 0:
            return b, 0, 1
        gcd, x1, y1 = extended_gcd(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        return gcd, x, y
    _, x, _ = extended_gcd(a % m, m)
    return (x % m + m) % m

def create_backup(k: int, w: int) -> tuple:
    """
    Create a backup of private key k using witness w
    Returns: (backup_key, witness_public)
    """
    k_prime = (k * w) % n  # Backup key (mod n!)
    W = w * G              # Witness public point (can be shared)
    return k_prime, W

def recover_key(k_prime: int, w: int) -> int:
    """
    Recover original private key from backup
    Requires: backup_key and original witness
    """
    w_inv = mod_inverse(w, n)  # Modular inverse of witness
    k = (k_prime * w_inv) % n  # Recovered private key
    return k

# === Example Usage ===
if __name__ == "__main__":
    # Your original private key (keep secret!)
    k = 0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
    
    # Generate a random witness (MUST back this up securely!)
    w = 0xfedcba0987654321fedcba0987654321fedcba0987654321fedcba0987654321
    
    # Create backup
    k_prime, W = create_backup(k, w)
    
    print(f"Original private key: {hex(k)}")
    print(f"Backup key (k'): {hex(k_prime)}")
    print(f"Witness public (W): {W}")
    
    # Later... recover the original key
    k_recovered = recover_key(k_prime, w)
    
    print(f"Recovered key: {hex(k_recovered)}")
    print(f"✅ Recovery successful: {k == k_recovered}")
    
    # Verify public key relationship
    K = k * G
    K_prime = k_prime * G
    K_from_w = w * K
    
    print(f"✅ Homomorphic property holds: {K_prime == K_from_w}")
```

### TypeScript/JavaScript Example

```typescript
/**
 * W-Path Key Backup and Recovery Example
 * Requires: npm install elliptic
 */
import { ec as EC } from 'elliptic';

const ec = new EC('secp256k1');
const n = ec.n!; // Curve order as BN.js instance

function modInverse(a: bigint, m: bigint): bigint {
    let [old_r, r] = [a, m];
    let [old_s, s] = [1n, 0n];
    
    while (r !== 0n) {
        const q = old_r / r;
        [old_r, r] = [r, old_r - q * r];
        [old_s, s] = [s, old_s - q * s];
    }
    
    return ((old_s % m) + m) % m;
}

function createBackup(kHex: string, wHex: string): { 
    backupKey: string; 
    witnessPublic: string 
} {
    const k = BigInt(kHex);
    const w = BigInt(wHex);
    const nBig = BigInt(n.toString(16));
    
    const kPrime = (k * w) % nBig;
    const W = ec.g.mul(w.toString(16));
    
    return {
        backupKey: kPrime.toString(16).padStart(64, '0'),
        witnessPublic: W.encode('hex', true)
    };
}

function recoverKey(backupKeyHex: string, wHex: string): string {
    const kPrime = BigInt(backupKeyHex);
    const w = BigInt(wHex);
    const nBig = BigInt(n.toString(16));
    
    const wInv = modInverse(w, nBig);
    const k = (kPrime * wInv) % nBig;
    
    return k.toString(16).padStart(64, '0');
}

// === Example Usage ===
const k = '0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef';
const w = '0xfedcba0987654321fedcba0987654321fedcba0987654321fedcba0987654321';

const { backupKey, witnessPublic } = createBackup(k, w);
console.log('Backup Key:', backupKey);
console.log('Witness Public:', witnessPublic);

const recovered = recoverKey(backupKey, w);
console.log('Recovered:', recovered);
console.log('✅ Match:', k.slice(2) === recovered);
```

---

## 🔒 Security Considerations

### What to Keep Secret

| Item | Sensitivity | Storage Recommendation |
|------|-------------|----------------------|
| Private Key `k` | **CRITICAL** | Hardware wallet, never exposed |
| Witness `w` | **CRITICAL** | Separate secure backup, different location |
| Backup Key `k'` | **HIGH** | Can be stored less securely than `k` |

### Security Best Practices

1. **Witness Generation**: Use cryptographically secure random number generation
2. **Witness Storage**: Store `w` separately from `k'` (different locations/media)
3. **Witness Size**: Ensure `w` is in range `[1, n-1]`
4. **Zero Knowledge**: The witness public `W = w·G` reveals no information about `w`

### Threat Model

```
┌─────────────────────────────────────────────────────────────┐
│                    COMPROMISE SCENARIOS                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Attacker has:           Can they recover k?   Risk Level  │
│  ─────────────           ──────────────────   ──────────   │
│  k' only                 NO                    ✅ Safe      │
│  k' + W                  NO                    ✅ Safe      │
│  w only                  NO                    ✅ Safe      │
│  k' + w                  YES                   ❌ COMPROMISED│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

⚠️ **Critical**: If both `k'` and `w` are compromised, the original key `k` can be recovered!

---

## 🎯 Use Cases

1. **Key Recovery Service** - Allow users to backup keys without trusting a single party
2. **Social Recovery** - Split `w` using Shamir's Secret Sharing among trusted contacts
3. **Cold Storage Backup** - Create paper backups of hardware wallet keys
4. **Enterprise Key Management** - Separate key custody from backup custody

---

## ❓ FAQ

### General Questions

**Q: What makes W-Path different from other backup methods?**
A: W-Path uses mathematical homomorphism, allowing the backup to be verified without revealing the original key. The backup key produces a deterministic public key relationship.

**Q: What blockchains are supported?**
A: Any blockchain using secp256k1, including Bitcoin, Ethereum, BSC, Polygon, Avalanche, and many others.

**Q: Can I use this with my existing wallet?**
A: Yes! Since W-Path works with any secp256k1 private key, it's compatible with existing wallets.

### Technical Questions

**Q: What if my witness `w` is larger than the curve order `n`?**
A: All operations are modular. `w mod n` is used automatically. However, for security, generate `w` in range `[1, n-1]`.

**Q: What happens if I lose my witness `w`?**
A: Without `w`, the original key `k` **cannot** be recovered from `k'`. This is by design - the witness is as critical as the key itself.

**Q: Can I create multiple backups with different witnesses?**
A: Yes! You can create backups `k'_1 = k·w_1`, `k'_2 = k·w_2`, etc. Each requires its corresponding witness to recover.

**Q: Is there a way to verify my backup is correct?**
A: Yes! Compute `K = k·G`, then verify `K' = k'·G = w·K`. If these match, your backup is valid.

---

## 🛠️ Troubleshooting

| Problem | Cause | Solution |
|---------|-------|----------|
| "Invalid scalar" error | `k` or `w` ≥ curve order | Ensure values are `< n` |
| Recovery produces wrong key | Modular inverse not applied | Use `k'·w⁻¹ mod n`, not just `k'/w` |
| Public key mismatch | Forgot modular arithmetic | All operations must be `mod n` |
| Integer overflow | Using regular integers | Use big integers (BigInt, BN.js) |

### Common Implementation Mistakes

```python
# ❌ WRONG: Integer division
k = k_prime / w

# ✅ CORRECT: Modular inverse
k = (k_prime * mod_inverse(w, n)) % n

# ❌ WRONG: Not using modular arithmetic
k_prime = k * w

# ✅ CORRECT: Modular multiplication
k_prime = (k * w) % n
```

---

## ✍️ Contributing

We welcome contributions! Here's how to help:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

### Ways to Contribute

- 📝 Improve documentation
- 🧪 Add test cases
- 💻 Add code examples in other languages (Rust, Go, C++)
- 🐛 Report bugs or issues
- 💡 Suggest new features

---

## 📚 References

- [SEC 2: Recommended Elliptic Curve Domain Parameters](https://www.secg.org/sec2-v2.pdf)
- [BIP-32: Hierarchical Deterministic Wallets](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
- [Elliptic Curve Cryptography - Wikipedia](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography)
- [ecdsa Python Library](https://github.com/warner/python-ecdsa)
- [elliptic JavaScript Library](https://github.com/indutny/elliptic)

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Questions?** Open an [issue](https://github.com/Crypto2Bank/wpath/issues) or contact the author.

**Found this useful?** ⭐ Star the repo!

</div>
