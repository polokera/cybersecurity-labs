# IPsec Site-to-Site VPN 

## Overview
This lab covers the configuration of a site-to-site IPsec VPN between two routers using Cisco packet tracer.

## Lab topology
Topology consists of two routers and two computers.
- R1: Local VPN endpoint
- R2: Remote VPN endpoint
- LAN 1: Network behind R1
- LAN 2: Network behind R2

## Command Breakdown
`access-list 110 permit ip 10.30.10.0 0.0.0.255 10.30.20.0 0.0.0.255`
 
### Enable security license 

```text
Router# configure terminal
Router(config)# license boot module c2900 technology-package securityk9
Router(config)# end
Router# copy running-config startup-config
Router# reload
```


```text
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 2
exit
```

### crypto isakmp policy 10
- These commands configure the IKE(Internet Key Exchange) policy.
- This is an agreement where devices agree on security parameters
- Before establishing IpSec 
- The command creates an IKE policy with a priority of 10.

### encr aes
- This specifies AES as the encryption algorithm

### authentication pre-share
- This specifies that the two routers will authenticate each other with a pre-shared secret key.
- For this Lab, the secret key is "LabOnly-VpnKey"

### group 2
- Specificies the Diffie-Hellman group used for the IKE policy. 
- Group 2 is weak for a production security.


```text
crypto isakmp key LabOnly-VpnKey address 198.51.100.2
crypto ipsec transform-set NSEC-SET esp-aes esp-sha-hmac
crypto map NSEC-MAP 10 ipsec-isakmp
 set peer 198.51.100.2
 set transform-set NSEC-SET
 match address 110
interface g0/1
 crypto map NSEC-MAP
end
```

### crypto isakmp key LabOnly-VpnKey address 198.51.100.2
- this tells the router that we are establishing an IKE relationship with the router at ip 198.51.100.2
- and the secret LabOnly-VpnKey will be used for authentication

### crypto ipsec transform-set NSEC-SET esp-aes esp-sha-hmac
- transform-set has the rules that define how to encrypt the data and how to chek that a packet has not been modified.
- transform-set [name] . NSEC-SET is just the name and it is arbitrary.
- *esp-aes* means that the encryption algorithm is aes.
- *esp-sha-hmac* provides integrity protection.

### crypto map NSEC-MAP 10 ipsec-isakmp
- crypto map tells the router how to use the configuration.
- the command creates crypto map name NSEC-MAP. This is the name of the whole policy we are building. 
- number 10 is the sequence number

### set peer 198.51.100.2
- this tells router1 "establish vpn with 198.51.100.2"

### set transform-set NSEC-SET
- this connects the crypto map with the transform set we created.

### match address 110
- this connects the crypto map to ACL 110.

### interface g0/1
- change to g0/1 interface so e can apply the crypto map on it

### crypto map NSEC-MAP

- tells the interface to use the traffic policy called NSEC-MAP on traffic passing through this interface

**this is then repeated on the other router, changing the IPs**

## Verification

- The VPN can be verified using commands such as:

```
show crypto isakmp sa
show crypto ipsec sa
```


 

