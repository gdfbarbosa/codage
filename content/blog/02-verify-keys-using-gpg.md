---
title: "Codage #2 - Verify your downloads using GPG keys"
date: "2023-01-13 11:15:01"
description: Verify your downloads using GPG keys
---

## Verify your downloads using GPG keys

As software engineers, we should verify identities of our downloads so we get some security if our files are damaged, changed or tampered with some dangerous data.

If you need to download Apache Kafka, or Apache Maven, you will find binaries in the site and the first thing you'll notice is that there are three files (downloaded file, checksum and signature):

```bash
apache-maven-3.8.7-bin.zip
apache-maven-3.8.7-bin.zip.sha512
apache-maven-3.8.7-bin.zip.asc
```

To check for the checksum, in SHA512, we can use:

`echo "$(cat apache-maven-3.8.7-bin.zip.sha512) apache-maven-3.8.7-bin.zip" | sha512sum -c`

And, well, for the signature it is quite more challenging.

Let's try:

```bash
gpg --verify apache-maven-3.8.7-bin.zip.asc apache-maven-3.8.7-bin.zip
```

And we get this output:

```
gpg: Signature made Sat Dec 24 16:21:41 2022 -03
gpg:                using RSA key 6A814B1F869C2BBEAB7CB7271A2A1C94BDE89688
gpg: Can't check signature: No public key
```

So, it seems like GPG is trying to check a signature but can't find the public key.

So, let's import it, using a keyserver well-know in [Apache Documentation](https://www.apache.org/info/verification.html):

`gpg --keyserver pgpkeys.mit.edu --recv-key 6A814B1F869C2BBEAB7CB7271A2A1C94BDE89688`

And we get this result:

```bash
gpg: key 1A2A1C94BDE89688: public key "Michael Osipov (Java developer) <1983-01-06@gmx.net>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Finally, we can try to verify the signature:

`gpg --verify apache-maven-3.8.7-bin.zip.asc apache-maven-3.8.7-bin.zip`

So, we get a message that it was verified, but we get a warning.

```bash
gpg: Signature made Sat Dec 24 16:21:41 2022 -03
gpg:                using RSA key 6A814B1F869C2BBEAB7CB7271A2A1C94BDE89688
gpg: Good signature from "Michael Osipov (Java developer) <1983-01-06@gmx.net>" [unknown]
gpg:                 aka "Michael Osipov <michaelo@apache.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 6A81 4B1F 869C 2BBE AB7C  B727 1A2A 1C94 BDE8 9688
```

This message typically means:

1. We do not trust this signature
2. We cannot certify that this signature belongs to the owner

For the first item, we can trust the signature using this command:

```bash
gpg --edit-key 6A814B1F869C2BBEAB7CB7271A2A1C94BDE89688
>sign
>yes
>save
```

And then, after verifying again we get this output:

```bash
gpg: Signature made Sat Dec 24 16:21:41 2022 -03
gpg:                using RSA key 6A814B1F869C2BBEAB7CB7271A2A1C94BDE89688
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   1  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:   1  signed:   0  trust: 1-, 0q, 0n, 0m, 0f, 0u
gpg: next trustdb check due at 2024-06-04
gpg: Good signature from "Michael Osipov (Java developer) <1983-01-06@gmx.net>" [full]
gpg:                 aka "Michael Osipov <michaelo@apache.org>" [full]
```

So, it seems right because we manually added this signature as trustworthy.

But that does not guarantee that it belongs to the user, because for that you should contact the owner and ask him to verify that this file is his own.

As per Apache Documentation, you should meet face-to-face to check that signature. We know this is quite impossible to do for each download, but the steps described above should at least provide some protection for your files.

Never skip these steps on your downloads! Better some protection than nothing.
