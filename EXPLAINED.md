# 🔐 AES-256 Explained — From Zero to Understanding

> **Who is this for?**
> This document is written for someone who has **zero knowledge** about encryption.
> No maths background needed. No coding experience needed.
> Just read it like a story.

---

## 📖 Table of Contents

1. [What is Encryption? (The Big Idea)](#1-what-is-encryption-the-big-idea)
2. [What is AES?](#2-what-is-aes)
3. [What does "256" mean in AES-256?](#3-what-does-256-mean-in-aes-256)
4. [The Key — Your Secret Password](#4-the-key--your-secret-password)
5. [The IV — Why the Same Message Looks Different Every Time](#5-the-iv--why-the-same-message-looks-different-every-time)
6. [CBC Mode — Chaining the Blocks](#6-cbc-mode--chaining-the-blocks)
7. [The 14 Rounds — What Actually Happens Inside AES](#7-the-14-rounds--what-actually-happens-inside-aes)
8. [Base64 — Making Encrypted Data Shareable](#8-base64--making-encrypted-data-shareable)
9. [Decryption — Running Everything in Reverse](#9-decryption--running-everything-in-reverse)
10. [What Happens with the Wrong Password?](#10-what-happens-with-the-wrong-password)
11. [How the Demo Page Works — Step by Step](#11-how-the-demo-page-works--step-by-step)
12. [Why AES-256 is Unbreakable (in Practice)](#12-why-aes-256-is-unbreakable-in-practice)
13. [Real-World Uses of AES-256](#13-real-world-uses-of-aes-256)
14. [Glossary — Quick Reference](#14-glossary--quick-reference)

---

## 1. What is Encryption? (The Big Idea)

Imagine you want to send a secret note to your friend, but a nosy person might read it on the way.
So you and your friend agree on a **secret code** — like replacing every letter with a number.

- `A = 1`, `B = 2`, `C = 3` … and so on.
- `"HELLO"` becomes `"8 5 12 12 15"`.

Only your friend (who knows the code) can read it.
Everyone else just sees random numbers.

**That is encryption** — turning readable information into unreadable scrambled data.

**Decryption** is the reverse — turning scrambled data back into the original message, using the same secret code.

> 🔑 The "secret code" in real encryption is called a **KEY**.

---

## 2. What is AES?

**AES** stands for **Advanced Encryption Standard**.

It is the encryption method used by:
- The **US Government** to protect classified information
- **WhatsApp, iMessage, Signal** to protect your chats
- **Banks** to protect your transactions
- **Your computer** when you lock a file or folder
- **HTTPS websites** (the lock icon in your browser)

AES was created in 1997, selected by the US National Institute of Standards and Technology (NIST) in 2001, and has **never been broken** since.

Think of AES as a **super-complex, uncrackable version** of the letter-to-number code from the example above — but involving 14 layers of complex mathematical shuffling.

---

## 3. What does "256" mean in AES-256?

The `256` refers to the **size of the key** — measured in **bits**.

### What is a bit?
A bit is the smallest unit of computer data. It can only be `0` or `1`.

- 1 bit = one switch (ON or OFF)
- 8 bits = 1 byte = one character (e.g., the letter `A`)
- 256 bits = 32 bytes = 32 characters

### Why does the key size matter?
The bigger the key, the more possible key combinations exist, and the harder it is for an attacker to **guess** the key.

| AES Version | Key Size | Possible Key Combinations |
|---|---|---|
| AES-128 | 128 bits | 340,000,000,000,000,000,000,000,000,000,000,000,000 |
| AES-192 | 192 bits | even more |
| AES-256 | 256 bits | **115 with 75 zeroes after it** |

Even if you had **every computer on Earth** trying every possible key, it would take **longer than the age of the universe** to crack AES-256.

> AES-256 is the **strongest version** of AES. It's what governments use for top-secret data.

---

## 4. The Key — Your Secret Password

In our demo, you type a **password** like `MySecretPassword123!`.

But AES needs a key that is **exactly 256 bits (32 bytes)**. Your password is probably not exactly 32 bytes. So we need to **convert it**.

### Key Derivation — Stretching Your Password

We use a process called **PBKDF2** (Password-Based Key Derivation Function 2).

Here's a simple analogy:

> Imagine putting your password through a **meat grinder 100,000 times**.
> What comes out is always exactly the same length (32 bytes),
> no matter how short or long the original password was.

**The process:**
```
Your Password  →  PBKDF2 (100,000 rounds of SHA-256 hashing)  →  256-bit Key
```

Why 100,000 rounds? Because a hacker trying to guess your password would have to do 100,000 rounds of computation **per guess** — making brute-force attacks extremely slow.

**SHA-256** (Secure Hash Algorithm 256) is the hashing function used. It takes any input and produces a fixed 32-byte "fingerprint". The same input always gives the same output, but it's impossible to go backwards — you can't get the original input from the fingerprint.

---

## 5. The IV — Why the Same Message Looks Different Every Time

**IV** stands for **Initialization Vector**.

Here's the problem it solves:

> If you encrypt `"Hello"` with the same key, and the result is always the same ciphertext,
> a hacker watching your messages could notice patterns:
> *"Every time they say hello, the ciphertext starts with X..."*

To prevent this, we generate a **random 16-byte number every time we encrypt** — the IV.
The IV is mixed into the encryption process, so even if the message and key are identical, the output is completely different.

**Analogy:**
> Think of the IV as a **random seed**. You and your friend use the same recipe (key), but you add a random ingredient (IV) that changes the dish completely — even if the base ingredients were the same.

The IV is **not secret** — it's stored alongside the ciphertext (prepended to it) so that the receiver can use it during decryption. Only the key needs to remain secret.

```
Encrypted Output = [16-byte IV] + [Encrypted Data]
```

---

## 6. CBC Mode — Chaining the Blocks

AES doesn't encrypt your entire message in one go. It works on **fixed-size chunks called blocks**, each exactly **128 bits (16 bytes)**.

**CBC** stands for **Cipher Block Chaining**.

### Why chaining?

If each block were encrypted independently, identical blocks of plaintext would produce identical blocks of ciphertext — a security weakness.

**Chaining** solves this:
- Block 1 is encrypted using the IV.
- Block 2 is encrypted using the **result of Block 1** as input.
- Block 3 is encrypted using the **result of Block 2** as input.
- And so on...

```
Plaintext:  [Block 1] [Block 2] [Block 3]
                ↓          ↓          ↓
          + IV      + Cipher1   + Cipher2
                ↓          ↓          ↓
Ciphertext: [Cipher1] [Cipher2] [Cipher3]
```

This means each encrypted block **depends on all previous blocks** — making the encryption much stronger and patterns impossible to detect.

---

## 7. The 14 Rounds — What Actually Happens Inside AES

AES-256 puts each 128-bit block through **14 rounds** of transformation. Think of each round as a layer of scrambling.

Each round does 4 operations:

---

### 7.1 SubBytes — The Substitution Box

Every single byte in the block is **replaced** by a different byte, using a lookup table called the **S-Box** (Substitution Box).

**Analogy:**
> It's like a secret codebook where every letter gets replaced with a completely different one.
> `A → Q`, `B → X`, `C → M` ... and so on.

This makes the output look completely unrelated to the input.

---

### 7.2 ShiftRows — The Shuffle

The block is arranged as a **4×4 grid of bytes**. Then the rows are shifted (rotated) by different amounts:

```
Before ShiftRows:       After ShiftRows:
[ a  b  c  d ]          [ a  b  c  d ]   ← row 0: no shift
[ e  f  g  h ]    →     [ f  g  h  e ]   ← row 1: shift left by 1
[ i  j  k  l ]          [ k  l  i  j ]   ← row 2: shift left by 2
[ m  n  o  p ]          [ p  m  n  o ]   ← row 3: shift left by 3
```

This **shuffles bytes across columns** so that each column now contains bytes from different parts of the original block.

---

### 7.3 MixColumns — The Mathematical Mix

Each column of the 4×4 grid is **mathematically multiplied** using special matrix math (Galois Field arithmetic — don't worry about the details!).

**Analogy:**
> Imagine blending all 4 ingredients in each column together in a blender — what comes out is a complete mix where every output byte depends on all 4 input bytes.

This is skipped in the very last round (round 14) because it would just add complexity without extra security.

---

### 7.4 AddRoundKey — XOR with the Subkey

The key you derived is also expanded into **15 subkeys** (one per round + initial). In this step, the current state of the block is **XOR-ed** with the subkey for that round.

**XOR (Exclusive OR)** is a simple binary operation:
- `0 XOR 0 = 0`
- `1 XOR 1 = 0`
- `0 XOR 1 = 1`
- `1 XOR 0 = 1`

**Analogy:**
> XOR is like a toggle switch. The key "flips" certain bits in your data. Only someone with the same key can "flip them back" to get the original.

---

### The Full Round Flow:

```
Start: Plaintext Block
  ↓
AddRoundKey (Round 0 — initial)
  ↓
┌─────────────────────────────────┐
│  SubBytes                       │
│  ShiftRows                      │  × 13 full rounds
│  MixColumns                     │
│  AddRoundKey                    │
└─────────────────────────────────┘
  ↓
SubBytes
ShiftRows
AddRoundKey  ← (Final round — no MixColumns)
  ↓
Output: Encrypted Block (Ciphertext)
```

After all 14 rounds, the block is completely unrecognisable from the original. This process repeats for **every 16-byte block** of your message.

---

## 8. Base64 — Making Encrypted Data Shareable

After encryption, the output is raw **binary data** — a sequence of bytes that may contain non-printable characters (things that look like garbage or cause issues in text fields, emails, URLs, etc.).

**Base64** converts binary data into a safe, text-friendly format using only 64 characters:
`A-Z`, `a-z`, `0-9`, `+`, `/`, and `=` for padding.

**Example:**
```
Raw bytes:  0x48 0xA3 0xFF 0x2C ...
Base64:     SKP/LA==...
```

This makes the ciphertext easy to copy, paste, send in a URL, or store in a database.

> Base64 is **not encryption** — it's just a way to represent binary data as text. Anyone can decode it. The security comes from the AES key, not Base64.

---

## 9. Decryption — Running Everything in Reverse

Decryption is simply the **reverse of encryption** — but you must have the **same key and IV**.

### Steps:
1. Take the Base64 string and decode it back to raw bytes
2. **Extract the IV** — the first 16 bytes of the decoded data
3. The remaining bytes are the **actual ciphertext**
4. Derive the same 256-bit key from the password (same PBKDF2 process)
5. Run AES in reverse through all 14 rounds:
   - **Inverse AddRoundKey**
   - **Inverse ShiftRows** (shift right instead of left)
   - **Inverse SubBytes** (use the inverse S-Box)
   - **Inverse MixColumns**
6. The output is the original plaintext — your message is back!

```
Ciphertext → [Inverse Round 14] → [Inverse Round 13] → ... → [Inverse Round 1] → Plaintext
```

The mathematics of AES are designed so that every operation has a **perfect inverse** — nothing is lost, the original data is perfectly recovered.

---

## 10. What Happens with the Wrong Password?

If you try to decrypt with the **wrong password**:

1. PBKDF2 derives a **completely different 256-bit key** (even one wrong character makes the key totally different)
2. AES runs the decryption with this wrong key
3. The output is **garbage** — random-looking bytes that mean nothing
4. Additionally, AES-CBC uses **PKCS#7 padding** — a padding scheme that tells the decryptor how many bytes were added to fill the last block. With the wrong key, the padding is invalid, and the operation throws an **error** — which is what you see in the demo

> This is a feature, not a bug! It means you can't accidentally decrypt to a "wrong but plausible" message — you either get the exact original, or you get a clear failure.

---

## 11. How the Demo Page Works — Step by Step

Here's exactly what happens when you press **Encrypt** in `demo.html`:

```
1. You type:  "Hello, Judges!"  (plaintext)
              "MyPassword123"   (password)

2. Browser calls: crypto.subtle.importKey()
   → Imports the password into the Web Crypto API

3. Browser calls: crypto.subtle.deriveKey() with PBKDF2
   → 100,000 rounds of SHA-256
   → Outputs: 32-byte (256-bit) AES key
   → Displayed as the hex grid of 32 coloured boxes on the page

4. Browser calls: crypto.getRandomValues(new Uint8Array(16))
   → Generates a cryptographically secure random 16-byte IV
   → Logged in the Process Log with its hex value

5. The AES pipeline animation plays (SubBytes → ShiftRows → ... )
   → This is just visual — the real computation is happening in parallel

6. Browser calls: crypto.subtle.encrypt({name:'AES-CBC', iv}, key, plaintext)
   → The Web Crypto API (built into your browser) runs the real AES-256 algorithm
   → Returns the raw encrypted bytes

7. We combine:  [16-byte IV] + [encrypted bytes]
   → Encode the whole thing as Base64
   → Display it in the "Ciphertext" box

8. The ciphertext is auto-pasted into the Decrypt section
```

When you press **Decrypt**:
```
1. Base64 string is decoded back to bytes
2. First 16 bytes  → IV
3. Remaining bytes → Ciphertext

4. Same PBKDF2 process derives the 256-bit key from the password
5. crypto.subtle.decrypt() runs AES in reverse
6. The original plaintext appears in the output box
7. ✔ or ✖ status is shown
```

> The browser's **Web Crypto API** (called `SubtleCrypto`) is a built-in cryptography library that ships with every modern browser. It uses the CPU's native AES hardware acceleration — meaning encryption happens in microseconds and is 100% secure.

---

## 12. Why AES-256 is Unbreakable (in Practice)

### Brute Force Attack
The most naive attack: just try every possible key until one works.

- AES-256 has **2²⁵⁶ possible keys**
- That's roughly **115,792,089,237,316,195,423,570,985,008,687,907,853,269,984,665,640,564,039,457,584,007,913,129,639,936** combinations
- Even if you had **1 trillion computers**, each trying **1 trillion keys per second**, it would take approximately **10⁵¹ years** — the age of the universe is only about **10¹⁰ years**.

### Mathematical Attacks
Cryptographers have studied AES for 25+ years and found **no practical mathematical shortcut** to break it. The best known theoretical attacks are millions of times too slow to be feasible.

### Side-Channel Attacks
These attack the *implementation* (e.g., measuring power consumption or timing), not AES itself. Our browser implementation is protected by the browser's secure crypto sandbox.

### The Real Threat: Humans, Not Math
In practice, AES-256 is never "broken" mathematically. The weaknesses are always:
- Weak passwords (easy to guess)
- Key stored insecurely (e.g., in a text file)
- Social engineering (someone tricks you into revealing the key)

> The math is perfect. The people are the weak link.

---

## 13. Real-World Uses of AES-256

| Where | How AES-256 is used |
|---|---|
| **WhatsApp / Signal** | Every message is encrypted before sending |
| **HTTPS (the lock icon)** | Your browser and the website use AES to encrypt the connection |
| **BitLocker / FileVault** | Full disk encryption on Windows/Mac |
| **Password Managers** | Your saved passwords are stored AES-256 encrypted |
| **US Military** | Classified "TOP SECRET" documents |
| **Bitcoin / Crypto wallets** | Private keys are protected with AES |
| **ZIP files with password** | Uses AES-256 (if you choose the right option) |
| **AWS / Google Cloud** | Data at rest is encrypted with AES-256 |

---

## 14. Glossary — Quick Reference

| Term | Plain English Meaning |
|---|---|
| **Plaintext** | The original readable message ("Hello World") |
| **Ciphertext** | The encrypted, unreadable scrambled output |
| **Key** | The secret used to lock and unlock the data |
| **AES** | The encryption algorithm — the set of rules for how scrambling works |
| **AES-256** | AES using a 256-bit key (the strongest variant) |
| **Bit** | A single 0 or 1 — the smallest unit of computer data |
| **Byte** | 8 bits. One character (like the letter 'A') is 1 byte |
| **256-bit key** | A key made of 256 bits = 32 bytes = 32 characters |
| **PBKDF2** | A function that converts your password into a fixed-size key |
| **SHA-256** | A one-way hashing function used inside PBKDF2 |
| **IV (Initialization Vector)** | A random number used to make each encryption unique |
| **CBC (Cipher Block Chaining)** | A mode where each encrypted block feeds into the next |
| **Block** | A 128-bit (16-byte) chunk that AES processes at a time |
| **Round** | One complete pass through SubBytes + ShiftRows + MixColumns + AddRoundKey |
| **SubBytes** | A byte-by-byte substitution using a fixed lookup table (S-Box) |
| **ShiftRows** | Rotating bytes within rows of the 4×4 grid |
| **MixColumns** | Mathematically mixing bytes within columns of the 4×4 grid |
| **AddRoundKey** | XOR-ing the block with a subkey derived from the main key |
| **XOR** | A binary operation that "toggles" bits — the key operation in AddRoundKey |
| **Base64** | A way to represent binary data as safe, printable text characters |
| **Web Crypto API** | A built-in browser library that performs real AES encryption in JavaScript |
| **Brute Force** | Trying every possible key — made impossible by the sheer size of AES-256 keys |
| **Padding (PKCS#7)** | Extra bytes added to make the last block exactly 16 bytes; validates decryption |
| **Key Derivation** | The process of turning a human-typed password into a machine-usable key |

---

## Quick Summary (TL;DR)

```
You type: "Hello Judges"  +  Password: "MyPassword"
                    ↓
    Password → PBKDF2 × 100,000 → 256-bit Key (32 bytes)
    Random IV (16 bytes) generated
                    ↓
    Message split into 16-byte blocks
    Each block XOR'd with IV (first) or previous ciphertext (CBC chaining)
                    ↓
    14 rounds of: SubBytes → ShiftRows → MixColumns → AddRoundKey
                    ↓
    Output: Raw encrypted bytes
                    ↓
    [IV bytes] + [Ciphertext bytes] → Base64 encode
                    ↓
    Final output: "U2FsdGVkX1+abc123..." (safe to copy/paste anywhere)
```

**To decrypt:** Give the Base64 string + the same password → run everything backwards → get "Hello Judges" back perfectly.

**With wrong password:** You get either garbage or an error. The original message is safe.

---

*Written for the AES-256 Live Demo project — `demo.html`*
*Encryption is not magic. It's maths. And the maths is beautiful.* 🔐
