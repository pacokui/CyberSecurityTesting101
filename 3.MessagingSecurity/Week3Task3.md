# Task 3: Digital Identity and Trust

In this task, we explored digital certificates, public-key cryptography, and the chain of trust using command-line tools `curl` and `openssl`.

## 1. Experimentation with `curl` and `openssl`

I selected **reddit.com** as the target for this experiment to analyze its secure connection setup.

**Commands executed:**

1. **Inspect the TLS handshake:**
   ```bash
   curl -v [https://reddit.com](https://reddit.com)
   Based on the output from the openssl command, here is the detailed analysis of the Reddit certificate:

Who has issued the certificate? The certificate was issued by DigiCert Inc.

Issuer: C=US, O=DigiCert Inc, CN=DigiCert Global G2 TLS RSA SHA256 2020 CA1

For what exact domain it has been certificated? It is a Wildcard Certificate issued for *.reddit.com. This means it is valid for reddit.com and all of its subdomains (e.g., www.reddit.com, old.reddit.com).

Subject: CN=*.reddit.com

When does it expire? The certificate is valid until May 22, 2026.

NotAfter: May 22 23:59:59 2026 GMT

What is the encryption algorithm and key size? The certificate uses RSA encryption with a 2048-bit key.

Details: Public key type RSA (2048 bit)

3. Manual Validation
To better understand how the "Chain of Trust" works, I manually verified the certificate locally on Kali Linux.

Steps taken:

I ran openssl s_client to get the raw certificates.

I saved the Server Certificate (the first block) to a file named reddit.crt.

I saved the Intermediate Certificate (the second block) to a file named intermediate.crt.

I ran the verification command, using the intermediate file to bridge the gap to my system's trusted root store.

Command:

Bash
openssl verify -untrusted intermediate.crt reddit.crt
Result: The command returned:

reddit.crt: OK

This confirms that the certificate is valid and chains back to a trusted root CA on my system.
