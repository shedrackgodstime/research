---
title: "What If We Just Put Files in the Database?"
summary: First session — starting from "files are bytes" and experimenting with base64, encryption, and compression
status: draft
portfolio: false
started: 2026-05
tags: [bytes, filesystems, encryption, compression, base64]
---

# What If We Just Put Files in the Database?

## Where It Started

Asked why people use S3 and separate file storage instead of dumping files in the database. Got told it's a performance/scale thing, not a "can't do it" thing. Wanted to see for myself.

## Session 1

Had a PNG file sitting in an empty repo. Didn't know where to start.

**xxd** shows the raw hex of a file. Ran it on the PNG and saw the magic bytes `89 50 4e 47` — that's "PNG" in ASCII. Never noticed that before.

If a file is bytes and bytes can turn into text, can I store a file as a text string and get it back?

**base64** turns binary into ASCII text. Ran it on the PNG:

```
base64 Mexico_coat_of_arms.png > portrait_as_text.txt
```

Opened the file. Looks like random characters. Then reversed it:

```
base64 -d portrait_as_text.txt > reconstructed.png
```

Compared with sha256sum. Both had the same hash. The bytes survived a round trip through plain text. So yeah, you can store a file as text and get it back perfectly.

What if someone steals the database? Encrypt first, then store.

```
openssl enc -aes-256-cbc -salt -in Mexico_coat_of_arms.png -out encrypted.enc -pass pass:"supersecret"
base64 encrypted.enc > encrypted_as_text.txt
```

The encrypted version looked like pure noise. No PNG header. No structure. Decrypted it back — same hash again.

What about compression? Can you squish a string and get back the exact same thing?

- Base64 text: 301KB -> 228KB with gzip (compresses fine, limited character set)
- Raw PNG: barely budged (already compressed)
- Encrypted file: didn't compress at all (encrypted data has no patterns for the compressor to find)

So compress before encrypting, not after.

## Still Want To

- Understand bytes at the bit level
- How compression algorithms actually work
- How encryption modes (CBC, GCM, CTR) differ at the byte level
- Maybe build a small system that stores files in a database as text

## Commands

```
xxd file.png
base64 file.png > text.txt
base64 -d text.txt > copy.png
sha256sum file1 file2

openssl enc -aes-256-cbc -salt -in file.png -out file.enc -pass pass:"key"
openssl enc -aes-256-cbc -d -in file.enc -out file.png -pass pass:"key"

gzip -c file > file.gz
gunzip -c file.gz > file
```

## References

- man xxd, man base64, man openssl
- PNG spec magic bytes
