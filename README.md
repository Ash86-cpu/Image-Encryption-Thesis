# Image Encryption: Cryptographic Procedures to Ensure Data Security

MSc thesis project (Cybersecurity Analytics, University of Exeter, 2023) comparing
symmetric and asymmetric encryption for protecting image files, measured on
memory use and run time.

- **Symmetric:** Blowfish with a random 128-bit key
- **Asymmetric:** RSA-2048 with OAEP padding, applied to the image in 214-byte segments

## Results

Across 15 forensic test runs, Blowfish used **32.6% less memory** than RSA
(1.29 GB vs 1.71 GB on average, a saving of 0.42 GB). Symmetric encryption is the
practical choice for bulk data such as images, while asymmetric encryption is
better suited to small payloads like keys.

## How it works

`Finalimageencryptiondecryption.py` is an interactive script:

| Mode | Encrypt | Decrypt |
|---|---|---|
| Symmetric (1) | Generates a random key, writes `<name>_blowfish_encrypted.png` and `encryption_key.bin` | Reads the encrypted file and the key, writes `<name>_blowfish_decrypted.png` |
| Asymmetric (2) | Generates an RSA-2048 key pair, writes `private_key.pem`, then encrypts the image segment by segment | Reads the encrypted file and `private_key.pem`, writes the decrypted image |

Each encrypt and decrypt function is decorated with `memory_profiler`'s
`@profile`, so a line-by-line memory report is printed as it runs, followed by
the total execution time.

## Requirements

- Python 3.8 or newer
- `pycryptodome`
- `memory-profiler`

```bash
pip install pycryptodome memory-profiler
```

## Usage

```bash
python Finalimageencryptiondecryption.py
```

Then follow the prompts: choose `1` (symmetric) or `2` (asymmetric), `E` to
encrypt or `D` to decrypt, and enter the file paths when asked.

## Limitations and what I would change today

This code was written to measure performance, not as a production tool. Looking
back at it with a security reviewer's eye:

- **ECB mode leaks patterns.** Blowfish runs in ECB mode, where identical blocks
  of input produce identical blocks of output. For images this is a real weakness:
  outlines of the original picture can remain visible in the encrypted file. CBC,
  CTR or GCM mode with a random IV would fix this.
- **Blowfish is dated.** Its 64-bit block size makes it vulnerable to birthday
  attacks on large volumes of data (Sweet32). AES is the modern standard.
- **RSA is not meant for bulk data.** Encrypting an image in hundreds of RSA
  segments is slow and memory-hungry, which is exactly what the results show.
  Real systems use hybrid encryption: a fast symmetric cipher for the data, with
  RSA protecting only the symmetric key.
- **Keys are stored unprotected** next to the encrypted files (`encryption_key.bin`,
  `private_key.pem`). In practice they would be password-protected or held in a
  key management system.
- **No integrity check.** Nothing detects whether an encrypted file was tampered
  with. An authenticated mode such as AES-GCM provides this.
