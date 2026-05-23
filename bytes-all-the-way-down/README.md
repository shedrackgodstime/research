---
title: "What If We Just Put Files in the Database?"
summary: First session - starting from "files are bytes" and experimenting with base64, encryption, and compression
status: active
portfolio: true
started: 2026-05
tags: [bytes, filesystems, encryption, compression, base64]
---

# What If We Just Put Files in the Database?

## Where It Started

I asked: *why do people use file storage (S3, etc.) instead of just putting files in the database?*

Got told it's about performance and scale, not about whether it's possible. That made me curious  can a file actually live as text in a database? So I started poking around.

## Session 1  What I Did

I had a PNG file in an empty repo. I didn't know where to start.

First thing I learned: `xxd` shows the raw bytes of a file.

```
xxd Mexico_coat_of_arms.png
```

Saw the PNG magic bytes: `89 50 4e 47...`  that spells "PNG" in ASCII. Never noticed that before.

Then I asked: if a file is just bytes, and bytes can be turned into text, can I store a file as a text string and get it back?

Someone showed me `base64`:

```bash
base64 Mexico_coat_of_arms.png > portrait_as_text.txt
```

That turned the whole image into readable ASCII characters. I opened it  looked like random letters and numbers. Then reversed it:

```bash
base64 -d portrait_as_text.txt > reconstructed.png
```

Checked if they were identical with `sha256sum`  both had the exact same hash. So the bytes survived a round trip through plain text. That was interesting.

Next I wondered: if I'm putting file bytes into a database, what if someone steals the database? Can I encrypt the file first, then store the encrypted version as text?

```bash
openssl enc -aes-256-cbc -salt \
    -in Mexico_coat_of_arms.png \
    -out encrypted.enc \
    -pass pass:"supersecret"

base64 encrypted.enc > encrypted_as_text.txt
```

Compared the original base64 text vs the encrypted base64 text  the encrypted version looked completely different. No structure, no PNG header, nothing. Decrypted it back and got the same hash again.

Then I asked about compression: can a string be squished smaller and still come back exactly the same?

Tried `gzip` on the base64 text  went from 301KB to 228KB. Tried it on the raw PNG  barely budged (PNGs are already compressed). Tried it on the encrypted file  didn't compress at all (encrypted data looks random, no patterns for the compressor to find).

That last part clicked: **compress before encrypt, not after.**

## Current State

The topic is active. The first session established the basic pipeline and the
main question now is how these byte-level ideas map to real storage systems,
compression internals, and encryption modes.

## What I Still Want to Understand

- How bytes actually work at the bit level
- How compression algorithms find patterns
- How different encryption modes (CBC, GCM, CTR) differ at the byte level
- Maybe build a tiny system that stores files as text in a database
- Everything else I'll discover as I go

## Raw Commands From This Session

```bash
xxd file.png
base64 file.png > text.txt
base64 -d text.txt > copy.png
sha256sum file1 file2

openssl enc -aes-256-cbc -salt -in file.png -out file.enc -pass pass:"key"
openssl enc -aes-256-cbc -d -in file.enc -out file.png -pass pass:"key"

gzip -c file > file.gz
gunzip -c file.gz > file
```

## References (So Far)

- `man xxd`
- `man base64`
- `man openssl`
- PNG spec  magic bytes section
