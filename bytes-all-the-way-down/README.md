---
title: "What If We Just Put Files in the Database?"
summary: A question-driven exploration of files as bytes — storage, encryption, compression, and why the line between files and databases is thinner than it looks
status: active
portfolio: false
started: 2026-05
tags: [bytes, filesystems, encryption, compression, base64]
---

# What If We Just Put Files in the Database?

## Summary

This started with a simple question: *why do we need separate file storage? Can't we just save files in the database?*

That sounds naive until you realize a file is just bytes, and a database column can hold bytes. So what actually stops us?

This is the thread that got pulled.

## The Question

> "Why not save files in a database instead of having separate file storage? Isn't that more costly?"

Short answer: you can. People do it all the time.

```sql
CREATE TABLE files (
    id SERIAL PRIMARY KEY,
    data BYTEA
);
```

The real answer is about *tradeoffs* — performance, backup size, query contention, CDN integration. But none of that invalidates the core insight: **a file is just bytes, and a database stores bytes.**

Which led to the next question.

## Can We Store a File as Text?

If a database stores text, and a file is bytes, can we turn bytes into text and back?

```bash
# Take a real PNG file
# Store it as text (like a DB text column would)
base64 Mexico_coat_of_arms.png > portrait_as_text.txt

# Convert it back
base64 -d portrait_as_text.txt > reconstructed.png

# Are they the same?
sha256sum Mexico_coat_of_arms.png reconstructed.png
```

```
4e5fbe0d55aa4be76429a8a26876132667f572014f408ae8e16972196a8151cd  original.png
4e5fbe0d55aa4be76429a8a26876132667f572014f408ae8e16972196a8151cd  reconstructed.png
```

**Identical.** The file lived as a text string, then came back byte-perfect.

## But What If the Database Gets Leaked?

If we're storing file contents directly — even as text — a DB dump means leaked files.

> "Can we encrypt the contents before putting them in the DB?"

```bash
# Encrypt the file
openssl enc -aes-256-cbc -salt \
    -in Mexico_coat_of_arms.png \
    -out encrypted.enc \
    -pass pass:"supersecret"

# Store as text (what the DB would hold)
base64 encrypted.enc > encrypted_as_text.txt

# Reverse later
base64 -d encrypted_as_text.txt > encrypted_from_db.enc
openssl enc -aes-256-cbc -d \
    -in encrypted_from_db.enc \
    -out decrypted.png \
    -pass pass:"supersecret"

sha256sum original.png decrypted.png  # match
```

The encrypted text looks like gibberish. Even with a full DB dump, the files are worthless without the key.

## Can We Squeeze It Smaller?

> "Can a string be compressed? Like, a long string into a short one, and get back the exact same thing?"

Yes — that's lossless compression. gzip, zlib, deflate, LZMA. They work by finding repeated patterns and replacing them with shorter references.

But there's a trap:

| Data | Before | After gzip | What happened |
|------|--------|------------|---------------|
| Raw encrypted bytes | 223K | 223K | Grew — encryption looks random, no patterns |
| Base64 text | 301K | 228K | Compressed — limited charset has patterns |
| Original PNG | 223K | 222K | Barely — PNG is already compressed |

**The rule:** compress *before* encrypting. Encryption destroys patterns. Compression needs patterns.

```
Correct:  File → Compress → Encrypt → Store
Wrong:    File → Encrypt → Compress → Store  ← wastes effort
```

## What It Adds Up To

A single pipeline covers all of it:

```
Binary file → gzip compress → AES-256 encrypt → base64 encode → store as text in DB
```

Reverse to retrieve. The file is compressed, encrypted, and stored where no filesystem expects it. And it works.

The line between "file storage" and "database" is thinner than it looks. At the bottom, everything is bytes.

## Next Steps (For Later)

- Deeper into binary: bits, bytes, endianness, how integers and floats actually look in memory
- How compression algorithms work under the hood (Huffman, LZ77)
- Encryption modes (CBC, GCM, CTR) and how they transform bytes
- Building a minimal file-in-DB system from scratch

## Commands Reference

```bash
# View raw hex
xxd file.png

# Store file as text / reconstruct
base64 file.png > text.txt
base64 -d text.txt > copy.png

# Encrypt / decrypt (AES-256-CBC)
openssl enc -aes-256-cbc -salt -in file.png -out file.enc -pass pass:"key"
openssl enc -aes-256-cbc -d -in file.enc -out file.png -pass pass:"key"

# Verify identity
sha256sum file1 file2

# Compress / decompress
gzip -c file > file.gz
gunzip -c file.gz > file
```

## References

- [PNG Specification — magic bytes explained](http://www.libpng.org/pub/png/spec/)
- OpenSSL `enc` man page
- Base64 encoding scheme (RFC 4648)
