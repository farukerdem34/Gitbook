---
description: Port 443 > 80
---

# PKI, TLS / SSL

This summarises what's the difference between HTTP and HTTPS, where the S stands for secure.&#x20;

## Public Key Infrastructure

PKI refers to the set of technologies, frameworks, standards and procedures that supports public key encryption and authentication. PKI for networks revolves around **asymmetric encryption**, which uses a mathematically generated public and private key, both of which are assigned to verify the identites of the endpoints (like Google.com, for example).&#x20;

{% embed url="https://rouvin.gitbook.io/pentesting/pentesting-methodology/terms-and-concepts/passwords-and-encryption#asymmetric-encryption" %}

In essence, PKI allows for end-to-end encryption, which would basically form a 'tunnel' from the host to the endpoint and protect sensitive information, and provide digital identites for users and devices. If we didn't have this, any attacker can simply intercept all the packets being transferred in the air and be able to raed sensitive information in plaintext.

### Terms

