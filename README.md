# 🔐 FortiGate IPsec VPN Configuration - IKEv2 (Mode-Config with EAP Authentication)
This configuration sets up a remote access IPsec VPN on FortiGate using IKEv2, EAP-based user authentication, and mode-config for dynamic IP assignment to clients. It supports split tunneling with custom DNS (Google DNS 8.8.8.8), assigns IPs from the 192.168.186.10–20 range, and allows access only to internal networks defined in specific range.

### Key features:

* IKEv2 with AES-256/SHA-256 encryption

* EAP authentication via user group

* Mode-config with client IP pool

* Split-tunnel access

* Static route for client subnet

## 1. Phase 1 Interface Configuration

```shell
config vpn ipsec phase1-interface
edit "SYSACCESS-IPSEC"
    set type dynamic                        # VPN peer has dynamic IP
    set interface "wan1"                    # Interface to listen for VPN connections
    set ike-version 2                       # Using IKEv2 for key exchange
    set peertype one                        # Accept connection from a single peer
    set net-device disable                  # Disables creation of a separate virtual interface for remote subnet
    set mode-cfg enable                     # Enable mode-config (client gets IP from FortiGate)
    set ipv4-dns-server1 8.8.8.8            # DNS server assigned to clients
    set proposal aes256-sha256              # Encryption and authentication proposal
    set dhgrp 14                            # Diffie-Hellman Group 14 (2048-bit)
    set eap enable                          # Enable EAP for authentication
    set eap-identity send-request           # FortiGate sends EAP identity request to client
    set authusrgrp "VPN-USER-GRP"           # Group containing allowed VPN users
    set peerid "vpn-user"                   # Expected peer identity (can be matched with client ID)
    set default-gw 192.168.186.1            # Default gateway for mode-config clients
    set ipv4-start-ip 192.168.186.10        # Start IP for client pool
    set ipv4-end-ip 192.168.186.20          # End IP for client pool
    set ipv4-netmask 255.255.255.0          # Subnet mask assigned to clients
    set ipv4-split-include "server_address" # Specifies split tunneling route (clients only access defined internal subnets)
    set psksecret *********                 # Encrypted pre-shared key
    set dpd-retryinterval 60                # Dead Peer Detection retry every 60 seconds
next
end
```
## 2. Phase 2 Interface Configuration
```shell
config vpn ipsec phase2-interface
edit "SYSACCESS-IPSEC-P2"
    set phase1name "SYSACCESS-IPSEC"       # Link Phase 2 to corresponding Phase 1
    set proposal aes256-sha256             # Phase 2 encryption/auth proposal
    set dhgrp 14                           # DH Group for Phase 2 (must match Phase 1)
next
end
```
## 3. Tunnel Interface Definition
```shell
config system interface
edit "SYSACCESS-IPSEC"
    set ip 192.168.186.1 255.255.255.255    # Assign a /32 IP to the tunnel interface
    set type tunnel                         # Marks this interface as a VPN tunnel
next
end
```
## 4. Firewall Policy for SYSACCESS-IPSEC VPN
```shell
config firewall policy
edit 1000
    set name "VPN_SYSACCESS_TO_INTERNAL"
    set srcintf "SYSACCESS-IPSEC"                 # Tunnel interface
    set dstintf "LAN"                             # Destination interface (e.g., internal network)
    set srcaddr "all"                             # Or define a specific address group if needed
    set dstaddr "server_Range"                    # Internal network/subnet
    set action accept
    set schedule "always"
    set service "ALL"                             # Or limit to specific services (HTTP, RDP, etc.)
    set nat enable                                # Enable NAT if clients need internet access
    set logtraffic all                            # Log all traffic for auditing
next
end
``` 

### Full Configuration
```shell
config vpn ipsec phase1-interface
edit "SYSACCESS-IPSEC"
        set type dynamic
        set interface "wan1"
        set ike-version 2
        set peertype one
        set net-device disable
        set mode-cfg enable
        set ipv4-dns-server1 8.8.8.8
        set proposal aes256-sha256
        set dhgrp 14
        set eap enable
        set eap-identity send-request
        set authusrgrp "VPN-USER-GRP"
        set peerid "vpn-user"
        set default-gw 192.168.186.1
        set ipv4-start-ip 192.168.186.10
        set ipv4-end-ip 192.168.186.20
        set ipv4-netmask 255.255.255.0
        set ipv4-split-include "server_address"
        set psksecret ***********
        set dpd-retryinterval 60
    next
end

config vpn ipsec phase2-interface
edit "SYSACCESS-IPSEC-P2"
        set phase1name "SYSACCESS-IPSEC"
        set proposal aes256-sha256
        set dhgrp 14
    next
end

config system interface
edit "SYSACCESS-IPSEC"
        set ip 192.168.186.1 255.255.255.255
    next
end
```
## forticlient configuration
![image](https://github.com/user-attachments/assets/c36e7a0f-e306-4c8c-979d-bfc98e97edb2)

![image](https://github.com/user-attachments/assets/b0420381-528b-4688-913f-af4eebd6be61)

![image](https://github.com/user-attachments/assets/88a9fd98-d75a-491e-ad6b-cd292e659a35)

![image](https://github.com/user-attachments/assets/5661267d-bc26-4a7c-8e16-eeafb682a675)

