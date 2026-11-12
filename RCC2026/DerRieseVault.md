# Der Riese Vault

RowdyCyberCon CTF 2026 · Crypto · Solved

# Challenge:

Recover a flag locked in the "Der Riese" vault. The entire solve is **offline** — no game state is changed and nothing is submitted back to the instance. The flag is derived by reconstructing a hash chain and using it to decrypt the ciphertext.

The vault key is the end of a 100-round hash chain:

- **Round 1 seed:** `state = sha256("DER_RIESE::MAINFRAME::115")`.
- **Advance:** for each round `r` from 2 through 100, `state = sha256(state + ":" + r)`. That's 99 iterations.
- **Key derivation:** `key = sha256(state + ":FLAG")`.

# Decryption:

The ciphertext is encrypted in 32-byte blocks with a keystream derived from the key:

- For block `i`, the keystream is `sha256(key + ":" + i)`.
- Plaintext is `ciphertext XOR keystream`, block by block.
- An integrity tag (`c582f09a…`) is checked against the decrypted plaintext, and it matched, confirming the chain and key were reconstructed correctly.

# Environment note:

The replay ran in PowerShell using .NET's SHA-256, because neither Node nor Python was installed on the machine at hand. Any language with a SHA-256 primitive reproduces it identically (the whole scheme is standard hashing and XOR).

## Takeaway:

A long hash chain looks intimidating but is fully deterministic: given the seed and the rule, every round is reproducible offline. The integrity tag is the useful tell, it lets you confirm you rebuilt the key correctly before trusting the decrypted output, instead of guessing whether a garbled result means a wrong key or a wrong method.
