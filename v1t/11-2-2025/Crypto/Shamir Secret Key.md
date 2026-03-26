Description:
*Each duck carries a small piece of a hidden message — alone it’s useless, but together the pieces form the whole. The secret was split into 6 shares and distributed: every participant holds one little piece. No single person can reconstruct the secret, but if at least three people combine their pieces, the original message appears.*

Solution:
We were given text file containing 5 people while need at least any 3 of them to get the secret key:

```text
Bob-ef73fe834623128e6f43cc923927b33350314b0d08eeb386
Sang-2c17367ded0cd22e15220a2b2a6cede16e2ed64d1898bbad
Khoi-e05fd9646ff27414510dec2e46032469cd60d632606c8181
Long-0c4de736ced3f8412307729b8bea56cc6dc74abce06a0373
Dung-afe15ff509b15eb48b0e9d72fc2285094f6233ec98914312
Steve-cb1a439f208aa76e89236cb496abaf20723191c188e23f54
```

**The idea behind Shamir’s Secret Sharing (SSS):**

Shamir’s Secret Sharing, created by **Adi Shamir** in 1979, is a **threshold-based secret sharing scheme**.
### 🔐 Goal
You want to **split a secret** into multiple parts (shares) such that:
- Any **k** of them can reconstruct the secret.
- But **fewer than k** reveal **absolutely nothing** about the secret.

All based on polynomials where each person represent itself in his/her order and his sharing as x-coordinate and y-coordinate point. `e.g. Bob is (1, ef73fe834623128e6f43cc923927b33350314b0d08eeb386)`

So you get a polynomial:
f(x) = a₀ + a₁x + a₂x² + ⋯ + aₖ₋₁xᵏ⁻¹ (mod p)
such that: a₀ is the secret and a₁...aₖ₋₁ are random coefficients that hide the secret.
We can get a₀ which the constant term from **Lagrange interpolation formula**:

      k
a₀ =   SUM  [ y_i * PRODUCT ( -x_j / (x_i - x_j) ) ]
      i=1          j≠i

Of course we won't solve it with our hands.

**Script:**

```python
import random

# -----------------------------
# Finite field operations (mod p)
# -----------------------------
PRIME = 2**127 - 1  # Large prime for modular arithmetic


def eval_polynomial(x, coefficients):
    """Evaluate polynomial (given coefficients) at x."""
    y = 0
    for i, coeff in enumerate(coefficients):
        y += coeff * pow(x, i, PRIME)
    return y % PRIME


# -----------------------------
# Split secret
# -----------------------------
def split_secret(secret, n, k):
    """Split integer secret into n shares with threshold k."""
    coeffs = [secret] + [random.randrange(0, PRIME) for _ in range(k - 1)]
    shares = [(i, eval_polynomial(i, coeffs)) for i in range(1, n + 1)]
    return shares


# -----------------------------
# Reconstruct secret
# -----------------------------
def reconstruct_secret(shares):
    """Reconstruct secret from (x, y) shares using Lagrange interpolation."""
    secret = 0
    for j, (xj, yj) in enumerate(shares):
        num, den = 1, 1
        for m, (xm, _) in enumerate(shares):
            if m != j:
                num = (num * (-xm)) % PRIME
                den = (den * (xj - xm)) % PRIME
        lagrange_coeff = (num * pow(den, -1, PRIME)) % PRIME
        secret = (secret + yj * lagrange_coeff) % PRIME
    return secret


# -----------------------------
# User interface
# -----------------------------
def main():
    print("\n🔐 Shamir's Secret Sharing (Interactive Tool)")
    print("=============================================")
    mode = input("Choose mode: [1] Split secret  [2] Reconstruct secret\n> ").strip()

    if mode == "1":
        # --- Split mode ---
        text_secret = input("\nEnter your secret (text): ").strip()
        secret_int = int.from_bytes(text_secret.encode(), "big")

        n = int(input("Number of total shares to generate (n): "))
        k = int(input("Minimum number of shares required to reconstruct (k): "))

        shares = split_secret(secret_int, n, k)

        print("\n✅ Generated Shares:")
        for x, y in shares:
            print(f"  Share {x}: {x}-{hex(y)[2:]}")

        print("\nStore these safely! Any", k, "of them can reconstruct the secret.")

    elif mode == "2":
        # --- Reconstruct mode ---
        print("\nEnter shares as 'x,y' (one per line). Type 'done' when finished.")
        shares = []
        while True:
            line = input("> ").strip()
            if line.lower() == "done":
                break
            try:
                x_str, y_str = line.split(",")
                x = int(x_str.strip())
                y = int(y_str.strip(), 0)  # auto-handle hex if prefixed with 0x
                shares.append((x, y))
            except Exception as e:
                print("⚠️ Invalid format. Use x,y (e.g., 1,0x1234abcd)")

        if len(shares) < 2:
            print("❌ Need at least 2 shares to reconstruct.")
            return

        recovered = reconstruct_secret(shares)
        recovered_bytes = recovered.to_bytes((recovered.bit_length() + 7) // 8, "big")

        print("\n✅ Reconstructed secret (int):", recovered)
        try:
            print("✅ Reconstructed secret (text):", recovered_bytes.decode())
        except UnicodeDecodeError:
            print("⚠️ Could not decode as text (binary data detected).")

    else:
        print("❌ Invalid option. Please choose 1 or 2.")


# -----------------------------
# Run
# -----------------------------
if __name__ == "__main__":
    main()

```

Flag:
**v1t{555_s3cr3t_sh4r1ng}**