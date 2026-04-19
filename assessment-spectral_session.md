# Assessment — The Spectral Session: LFI + Cryptographic Forgery + Insecure Deserialization

---

## Question 1 — MCQ

**What URL encoding trick bypasses the file endpoint's traversal filter to read `config.py`?**

- A) Double URL-encoding: `%252e%252e%252f`
- B) Null byte injection: `config.py%00.jpg`
- C) **URL-encoded backslash: `..%5c..%5cconfig.py`** ✅
- D) Unicode normalization: `..%c0%afconfig.py`

> **Answer:** C — The Flask router blocks standard `../` but doesn't normalise URL-encoded backslashes, allowing `%5c` to be resolved as a path separator server-side.

---

## Question 2 — MCQ

**What hashing algorithm and formula generate the Vault access token?**

- A) SHA-256 of `username + timestamp`
- B) HMAC-SHA1 of `user_id` using the server secret
- C) **MD5 of `user_id + VAULT_SALT`** ✅
- D) SHA-512 of `session_id + secret`

> **Answer:** C — The leaked `config.py` reveals `hashlib.md5(user_id + VAULT_SALT)` as the token formula, making it fully forgeable offline.

---

## Question 3 — MCQ

**What is the forged Vault token for user `usr_alice_001` with salt `sp3ctral_s3cr3t_2024`?**

- A) `a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6`
- B) `f7e6d5c4b3a2918273645566778899aa`
- C) **`d8b96cf9f4e6182e2056ae3f9bef9eaf`** ✅
- D) `0a1b2c3d4e5f67890abcdef012345678`

> **Answer:** C — `hashlib.md5(b"usr_alice_001sp3ctral_s3cr3t_2024").hexdigest()` produces this value.

---

## Question 4 — Fill in the Blank

**What is the `VAULT_SALT` value leaked from `config.py` via the path traversal exploit?**

**Answer:** `sp3ctral_s3cr3t_2024`

> This salt is discovered by reading `config.py` via `GET /files/..%5c..%5cconfig.py`. Combined with the known user ID `usr_alice_001`, it allows offline computation of the valid Vault token as `md5(usr_alice_001sp3ctral_s3cr3t_2024)`.

---

## Question 5 — Fill in the Blank

**What is the forged Vault access token for user ID `usr_alice_001`?**

**Answer:** `d8b96cf9f4e6182e2056ae3f9bef9eaf`

> Computed as `hashlib.md5(b"usr_alice_001sp3ctral_s3cr3t_2024").hexdigest()`. This token is accepted by `GET /vault?token=d8b96cf9f4e6182e2056ae3f9bef9eaf`, granting `can_vault` permissions and unlocking the archive endpoints.

---

*Lab target:* `http://localhost:5000`
