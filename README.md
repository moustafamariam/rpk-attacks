# rpk-attacks

# About
---
This repository contains the ProVerif models of the SSH, IPsec, and TLS protocols discussed in our paper "Attacks on Key Ownership in Common Security Protocols". The ProVerif models show the attack that can happen when no proper proof-of-possession is done before, during, or after the protocol. 

# Models
---

### SSH

- [Attack on SSH](SSH/SSH-SSHFP.pv): This model shows the redirection attack on SSH. It also contains an alternative value for the hash H that, when used, the attack fails.
- [SSH with proof-of-possession solution](SSH/SSH-SSHFP-PoP.pv): This is a model of the solution where an SSH server signs the DNS update request using the key in the resource record to verify its ownership.
- [SSH with self-signed certificates solution](SSH/SSH-SSHFP-SS.pv): This is a model of the solution where the SSH server sends its self-signed certificate to the client.

### IPsec

- [Attack on IPsec](IKEv2/IKEv2-IPSECKEY.pv): This model of IKEv2 shows the redirection attack on IPsec. It also contains a commented line of code where the responder verifies the received identity IDr. This line can be a potential solution.
- [IKEv2 with proof-of-possession solution](IKEv2/IKEv2-IPSECKEY-PoP.pv): This is a model of the solution where a responder signs the DNS update request using the key in the resource record to verify its ownership. 
- [IKEv2 with self-signed certificates solution](IKEv2/IKEv2-IPSECKEY-SS.pv): This is a model of the solution where the responder sends its self-signed certificate to the initiator.


### TLS RPK

- [Attack on TLS RPK](TLS-RPK/TLS-RPK-DANE.pv): This model shows the redirection attack on TLS RPK. It also contains a commented line of code where the client sends the *server_name* extension in the *ClientHello*. Sending this extension and the server verifying it, can be a potential solution.
- [TLS RPK with proof-of-possession solution](TLS-RPK/TLS-RPK-DANE-PoP.pv): This is a model of the solution where the server signs the DNS update request using the key in the resource record to verify its ownership. 
- [TLS with self-signed certificates solution](TLS-RPK/TLS-RPK-DANE-SS.pv): This is a model of the solution where the server sends its self-signed certificate to the client.

### Attack traces
- The [attack-traces](attack-trace/) folder contains the attacks detected by Proverif for the protocols. ProVerif was able to find attack traces for the attacks implemented in the paper.
