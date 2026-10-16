# rpk-attacks

# About
---
This repository contains the artefacts associated with our paper "Attacks on Key Ownership in Security Protocols". A documentation of how the proof of concept attacks were carried out in our testbed can be found in [attacks-PoC.md](attacks-PoC.md). The ProVerif models of the SSH, IPsec, and TLS protocols that also confirm the redirection attacks as well as the proposed solutions can be found in the protocol's corresponding folder name. The ProVerif models show that the attacks can happen when no proper proof-of-possession is done before, during, or after the protocol. 

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
- The [attack-traces](attack-traces/) folder contains the attacks detected by Proverif for the protocols. ProVerif was able to find attack traces for the attacks implemented in the paper.


# Citation
---

If you find this work useful in any way for your research, please cite our paper:

**Text:**

```text
M. Moustafa, M. Sethi, and T. Aura, "Attacks on Key Ownership in Security Protocols," in Computer Security – ESORICS 2026, 
Lecture Notes in Computer Science, Springer Nature Switzerland, 2026, pp. 439–459. doi: 10.1007/978-3-032-38695-3_22
```

**BibTeX:**

```bibtex
@inproceedings{moustafa2026keyownership,
  title     = {Attacks on Key Ownership in Security Protocols},
  author    = {Moustafa, Mariam and Sethi, Mohit and Aura, Tuomas},
  booktitle = {Computer Security -- ESORICS 2026},
  series    = {Lecture Notes in Computer Science},
  pages     = {439--459},
  year      = {2026},
  publisher = {Springer Nature Switzerland},
  doi       = {10.1007/978-3-032-38695-3_22},
  url       = {https://doi.org/10.1007/978-3-032-38695-3_22}
}
```