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
        set interface "wan1"                
        set type dynamic                     
        set ike-version 2
        set peertype any
        set mode-cfg enable                  
        set net-device disable               
        set add-route enable
        set proposal aes256-sha256
        set dhgrp 14                         
        set keylife 28800
        set authmethod psk
        set psksecret **************   
        set eap enable                       
        set eap-identity send-request        
        set authusrgrp ipsec_user_group     
        set ipv4-start-ip 192.168.189.10        
        set ipv4-end-ip   192.168.189.20
        set ipv4-netmask  255.255.255.0
        set dns-mode manual
        set ipv4-dns-server1 172.20.0.10   
        set ipv4-dns-server2 172.20.0.11
        set dpd on-idle
	    set localid "ikev2user"
        set nattraversal enable
        set comments "FortiClient IKEv2 remote access"
    next
end
```
## 2. Phase 2 Interface Configuration

```shell
config vpn ipsec phase2-interface
    edit "SYSACCESS-IPSEC-P2"
        set phase1name "SYSACCESS-IPSEC"
        set proposal aes256-sha256
        set pfs disable
        set keylifeseconds 3600
        set src-subnet 0.0.0.0 0.0.0.0
        set dst-subnet 0.0.0.0 0.0.0.0
    next
end
```
## 3. Firewall Policy for SYSACCESS-IPSEC VPN
```shell
config firewall policy
edit 1000
    set name "VPN_SYSACCESS_TO_INTERNAL"
    set srcintf "SYSACCESS-IPSEC"                  
    set dstintf "LAN"                             
    set srcaddr "all"                             
    set dstaddr "server_Range"                    
    set action accept
    set schedule "always"
    set service "ALL"                             
    set nat enable                                
    set logtraffic all                            
next
end
``` 

### Full Configuration

config vpn ipsec phase1-interface
    edit "SYSACCESS-IPSEC"
        set interface "wan1"                
        set type dynamic                     
        set ike-version 2
        set peertype any
        set mode-cfg enable                  
        set net-device disable               
        set add-route enable
        set proposal aes256-sha256
        set dhgrp 14                         
        set keylife 28800
        set authmethod psk
        set psksecret **************   
        set eap enable                       
        set eap-identity send-request        
        set authusrgrp ipsec_user_group     
        set ipv4-start-ip 192.168.189.10        
        set ipv4-end-ip   192.168.189.20
        set ipv4-netmask  255.255.255.0
        set dns-mode manual
        set ipv4-dns-server1 172.20.0.10   
        set ipv4-dns-server2 172.20.0.11
        set dpd on-idle
	set localid "ikev2user"
        set nattraversal enable
        set comments "FortiClient IKEv2 remote access"
    next
end


config vpn ipsec phase2-interface
    edit "SYSACCESS-IPSEC-P2"
        set phase1name "SYSACCESS-IPSEC"
        set proposal aes256-sha256
        set pfs disable
        set keylifeseconds 3600
        set src-subnet 0.0.0.0 0.0.0.0
        set dst-subnet 0.0.0.0 0.0.0.0
    next
end

```
## forticlient configuration
![image](https://github.com/user-attachments/assets/c36e7a0f-e306-4c8c-979d-bfc98e97edb2)

![image](https://github.com/user-attachments/assets/b0420381-528b-4688-913f-af4eebd6be61)

![image](https://github.com/user-attachments/assets/88a9fd98-d75a-491e-ad6b-cd292e659a35)

![image](https://github.com/user-attachments/assets/5661267d-bc26-4a7c-8e16-eeafb682a675)

