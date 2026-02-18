
<!-- Your Monitor Number == #$34T# -->


## Prereq
### Setup Virtual Network Adapters
`VMWare` > `Edit` > `Virtual Network Editor`  

<br>

Add/Edit the following VMNets:  

| VMNet 2    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.102.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 3    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.103.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 4    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.104.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 8    |               |
| ---        | ---           |
| VMNet Info | NAT           |
| IP address | 208.8.8.0     |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 	

<br>

| VMNet 15   |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 10.255.10.0   |
| Net Mask   | 255.255.255.0 |
| DHCP       | Checked       | 

<br>

| VMNet 16   |                 |
| ---        | ---             |
| VMNet Info | Host-only       |
| IP address | 10.69.255.0     |
| Net Mask   | 255.255.255.248 |
| DHCP       | Checked         | 


&nbsp;
---
&nbsp;


## Lab Setup

### 1. Deploy

Devices:
- 2x CSR1000v
- 2x TinyCore (yvm.ova)
- NetOps  


<br>


| VM        | NetAdapter | NetAdapter 2 | NetAdapter 3 | NetAdapter 4  |
| ---       | ---        | ---          | ---          | ---           |
| UTM-PH    | NAT        | VMNet2       | VMNet3       | Bridged (Rep) |
|           |            |              |              |               |
| UTM-JP    | NAT        | VMNet2       | VMNet4       |               |
|           |            |              |              |               |
| NetOps-PH | VMNet1     | VMNet2       | VMNet3       | Bridged (Rep) |
|           |            |              |              |               |
| BLDG-PH   | VMNet3     | VMNet2       |              |               |
|           |            |              |              |               |
| BLDG-JP-1 | VMNet4     |              |              |               |
|           |            |              |              |               |
| BLDG-JP-2 | VMNet4     |              |              |               |
|           |            |              |              |               |


&nbsp;
---
&nbsp;


### 2. Bootstrap
~~~
!@UTM-PH
conf t
 hostname UTM-PH
 enable secret pass
 service password-encryption
 no logging cons
 no ip domain lookup
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.11 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.11 255.255.255.0
  no shut
 int g3
  ip add 11.11.11.113 255.255.255.224
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 end
wr
!
~~~

<br>

~~~
!@UTM-JP
conf t
 hostname UTM-JP
 enable secret pass
 service password-encryption
 no logging cons
 no ip domain lookup
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.12 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.12 255.255.255.0
  no shut
 int g3
  ip add 21.21.21.213 255.255.255.240
  ip add 22.22.22.223 255.255.255.192 secondary
  no shut
  exit
 !
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 ip domain lookup
 ip name-server 8.8.8.8 1.1.1.1
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 end
wr
!
~~~

<br>

~~~
!@BLDG-PH-1
sudo su
ifconfig eth0 11.11.11.100 netmask 255.255.255.224 up
ifconfig eth1 192.168.102.100 netmask 255.255.255.0 up
route add default gw 11.11.11.113
ping 11.11.11.113
~~~

Create a user account:
~~~
!@BLDG-PH-1
adduser admin

> pass
> pass
~~~

<br>

~~~
!@BLDG-JP-1
sudo su
ifconfig eth0 21.21.21.211 netmask 255.255.255.240 up
route add default gw 21.21.21.213
ping 21.21.21.213
~~~

<br>

~~~
!@BLDG-JP-2
sudo su
ifconfig eth0 22.22.22.221 netmask 255.255.255.192 up
route add default gw 22.22.22.223
ping 22.22.22.223
~~~


<br>
<br>


## NetOps-PH Setup
> Login: root
> Pass: C1sc0123

<br>

### 1. Get the MAC Address for the Bridge connection
VMWare > NetOps-PH Settings > NetAdapter (2, 3, & 4) > Advance > MAC Address

| NetAdapter   | MAC Address      | VM Interface |           |
| ---          | ---              | ---          | ---       |
| NetAdapter 2 | ___.___.___.___  | ens___       |  ens192   |
| NetAdapter 3 | ___.___.___.___  | ens___       |  ens224   |
| NetAdapter 4 | ___.___.___.___  | ens___       |  ens256   |


<br>


### 2. Get Network-VM Mapping

~~~
!@NetOps-PH
ip -br link
~~~


<br>


### 3. Modify Interface IP

__Using Network Management CLI for persistent IP.__

~~~
!@NetOps-PH
nmcli connection add \
type ethernet \
con-name VMNET2 \
ifname ens192 \
ipv4.method manual \
ipv4.addresses 192.168.102.6/24 \
autoconnect yes

nmcli connection up VMNET2


nmcli connection add \
type ethernet \
con-name VMNET3 \
ifname ens224 \
ipv4.method manual \
ipv4.addresses 11.11.11.100/27 \
autoconnect yes

nmcli connection up VMNET3


nmcli connection add \
type ethernet \
con-name BRIDGED \
ifname ens256 \
ipv4.method manual \
ipv4.addresses 10.#$34T#.1.6/24 \
autoconnect yes

nmcli connection up BRIDGED


ip route add 10.0.0.0/8 via 10.#$34T#.1.4 dev ens256
ip route add 200.0.0.0/24 via 10.#$34T#.1.4 dev ens256
ip route add 0.0.0.0/0 via 11.11.11.113 dev ens224
~~~


<br>
<br>

---
&nbsp; 


### Access WEB GUI
Open a browser and enter the Gig2 IP address of the VPN.  

- http://192.168.102.11/

- http://192.168.102.12/


<br>
<br>

---
&nbsp;


# Certificates

## CA Hierarchy

1. ROOT CA
   - X509v3
   - Basic Constraints [Critical]
       CA
   - Key Usage [Critical]
       Certificate Sign
       CRL Sign

<br>


2. SUB CA
   - X509v3
   - Basic Constraints [Critical]
       CA
	   Path Len: 0
   - Key Usage [Critical]
       Certificate Sign
       CRL Sign

<br>


3. LEAF CA
   - X509v3
   - Basic Constraints [Critical]
       END-ENTITY
   - Extensions
       Subject Alt Names
   - Key Usage [Critical]
       Digital Signature
	   Key Encipherment
	   Data Encipherment
	   Key Agreement
   - Extended Key Usage
       TLS Web Server Authentication
       TLS Web Client Authentication	   
	   E-mail Protection
	   IPSec End System
	   IPSec Tunnel
	   IPSec User
	   IP Security end entity


&nbsp; 
---
&nbsp; 


## Certificates via OPENSSL

__IF USING TINYCORE FOR CA GENERATION__
~~~
!@BLDG-PH
mkdir certs; cd certs
~~~

<br>

~~~
!@BLDG-PH
vi /etc/resolve.conf


nameserver 8.8.8.8
~~~

<br>

EXIT OUT OF SUDO
~~~
!@BLDG-PH
tce-load -wi nano
echo nano.tcz >> /mnt/sda1/tce/onboot.lst
~~~

<br>
<br>


### STEP 1 - Creating Private Keys

__ROOT CA__
~~~
!@Linux
openssl genrsa -aes256 -out ca.key 2048
~~~

<br>

__INTERMEDIATE CA__
~~~
!@Linux
openssl genrsa -aes256 -out subca.key 2048
~~~


<br>
<br>

---
&nbsp;

### STEP 2 - Create an OPENSSL Configuration File for both the __ROOT CA__ & __INTERMEDIATE CA__

__ROOT CA__
~~~
!@Linux
nano ca.cnf
~~~

<br>

~~~
[ req ]
default_bits       = 2048
default_md         = sha256
prompt             = no
distinguished_name = dn
x509_extensions    = v3_ca


[ dn ]
C  = PH
ST = NCR
L  = Manila
O  = Rivancorp
OU = HQ
CN = Rivancorp Root CA


[ v3_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:TRUE
keyUsage               = critical, keyCertSign, cRLSign
~~~


<br>
<br>


__INTERMEDIATE CA__
~~~
!@Linux
nano subca.cnf
~~~

<br>

~~~
[ req ]
default_bits       = 2048
default_md         = sha256
prompt             = no
distinguished_name = dn
x509_extensions    = v3_subca


[ dn ]
C  = PH
ST = NCR
L  = Makati
O  = Rivancorp
OU = Makati Branch
CN = Rivancorp Intermediate CA


[ v3_subca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = critical, CA:TRUE, pathlen:0
keyUsage               = critical, keyCertSign, cRLSign
~~~


<br>
<br>

---
&nbsp;

### STEP 3 - Output the __ROOT CA__

__ROOT CA__
~~~
!@Linux
openssl req \
  -new -x509 \
  -key ca.key \
  -out ca.crt \
  -days 3650 \
  -extensions v3_ca \
  -config ca.cnf
~~~


<br>
<br>

---
&nbsp;

### STEP 4 - Generate CSR for the __INTERMEDIATE CA__

__INTERMEDIATE CA__
~~~
!@Linux
openssl req \
  -new \
  -key subca.key \
  -out subca.csr
~~~

<br>

~~~
Country Name (2 letter code) [XX]:                             PH
State or Province Name (full name) []:                         NCR
Locality Name (eg, city) [Default City]:                       Makati
Organization Name (eg, company) [Default Company Ltd]:         Rivancorp
Organizational Unit Name (eg, section) []:                     Makati Branch
Common Name (eg, your name or your server's hostname) []:      Rivancorp Intermediate CA


A challenge password []:                                       pass
An optional company name []:                                   
~~~


<br>
<br>

---
&nbsp;

### STEP 5 - Sign the CSR using the __ROOT CA__

~~~
!@Linux
openssl x509 \
  -req \
  -in subca.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -out subca.crt \
  -days 365 \
  -extensions v3_subca \
  -extfile subca.cnf
~~~


<br>
<br>

---
&nbsp;

### STEP 6 - Install the __ROOT CA__ & __INTERMEDIATE CA__ on Devices

__LINUX__
~~~
!@Linux
cp  ca.crt    /etc/pki/ca-trust/source/anchors/
cp  subca.crt    /etc/pki/ca-trust/source/
~~~

<br>

~~~
!@Linux
update-ca-trust enable
update-ca-trust
~~~

<br>


Verify
~~~
!@Linux
trust list | grep -i "Rivancorp"

openssl x509 -in utmph.crt -noout -issuer -subject
~~~


<br>
<br>


__WINDOWS__
~~~
!@Run
certlm.msc
~~~

<br>

Certificates 
  > Trusted Root Certification Authorities (Right-Click) 
    > All Tasks 
	  > Import

<br>

Certificates 
  > Intermediate Certification Authorities (Right-Click) 
    > All Tasks 
	  > Import


<br>
<br>


__CISCO__
~~~
!@Cisco
conf t
 crypto pki trustpoint RIVAN-CA
  enrollment terminal
  revocation-check crl none
  exit
 !
 crypto pki authenticate RIVAN-CA

 > Paste the CA
~~~


<br>
<br>

---
&nbsp;


### STEP 7 - Generate __LEAF CERTS__ or __END ENTITY CERTS__

__LINUX__  
~~~
!@Linux
openssl genrsa -aes256 -out utmph.key 2048
~~~

<br>

~~~
!@Linux
nano utmph.cnf 
~~~

<br>

~~~
[ req ]
default_bits       = 2048
default_md         = sha256
distinguished_name = dn
req_extensions     = v3_leaf_req
prompt             = no

[ dn ]
C  = PH
ST = NCR
L  = Makati
O  = Rivancorp
OU = Makati Branch
CN = ph.rivancorp.com

[ v3_leaf_req ]
basicConstraints    = critical, CA:false
keyUsage            = critical, digitalSignature, keyEncipherment
extendedKeyUsage    = serverAuth, clientAuth, ipsecEndSystem, ipsecTunnel, ipsecUser, ipsecIKE
subjectAltName      = @alt_names

[ alt_names ]
DNS.1   = utmph.rivancorp.com
IP.1    = 208.8.8.11
~~~

<br>

~~~
!@Linux
openssl req \
  -new \
  -key utmph.key \
  -out utmph.csr \
  -config utmph.cnf
~~~

<br>

~~~
!@NetOps
openssl x509 \
  -req \
  -in utmph.csr \
  -CA subca.crt \
  -CAkey subca.key \
  -CAcreateserial \
  -out utmph.crt \
  -days 30 \
  -extensions v3_leaf_req \
  -extfile utmph.cnf
~~~


<br>
<br>

---
&nbsp;


__WINDOWS__
~~~
!@Run
certlm.msc
~~~

<br>

Certificates   
  > Personal (Right-Click)   
    > All Tasks   
	  > Advance Operations   
	    > Create Custom Request  


<br>
<br>

---
&nbsp;


__CISCO__

~~~
!@UTM-PH
conf t
 crypto key generate rsa modulus 2048 label RIVANPH-KEY exportable
 end
~~~

<br>

~~~
!@UTM-PH
conf t
 crypto pki trustpoint RIVAN-PH
  enrollment terminal
  revocation-check crl none
  rsakeypair RIVANPH-KEY
  exit
 !
 
 
 crypto pki authenticate RIVAN-PH
 
 
 crypto pki enroll RIVAN-PH
 
~~~

<br>


-----BEGIN CERTIFICATE REQUEST-----

-----END CERTIFICATE REQUEST-----


<br>


~~~
!@NetOps
openssl x509 \
  -req \
  -in utmph.csr \
  -CA subca.crt \
  -CAkey subca.key \
  -CAcreateserial \
  -out utmph.crt \
  -days 30 \
  -extensions v3_leaf_req \
  -extfile utmph.cnf
~~~


<br>


~~~
!@UTM-PH
conf t
 crypto pki import RIVAN-PH certificate
 
~~~


<br>
<br>

---
&nbsp;


### ACTIVITY - Issue A Certificate for RIVAN-JP

~~~
!@Linux
nano utmjp.cnf 
~~~

<br>

~~~
[ req ]
default_bits       = 2048
default_md         = sha256
distinguished_name = dn
req_extensions     = v3_leaf_req
prompt             = no

[ dn ]
C  = JP
ST = Kanto
L  = Tokyo
O  = Rivancorp
OU = Tokyo Branch
CN = jp.rivancorp.com

[ v3_leaf_req ]
basicConstraints    = critical, CA:false
keyUsage            = critical, digitalSignature, keyEncipherment
extendedKeyUsage    = serverAuth, clientAuth, ipsecEndSystem, ipsecTunnel, ipsecUser, ipsecIKE
subjectAltName      = @alt_names

[ alt_names ]
DNS.1   = utmjp.rivancorp.com
IP.1    = 208.8.8.12
~~~


~~~
!@UTM-JP
conf t
 crypto key generate rsa modulus 2048 label RIVANJP-KEY exportable
 end
~~~

<br>

~~~
!@UTM-JP
conf t
 crypto pki trustpoint RIVAN-JP
  enrollment terminal
  revocation-check crl none
  rsakeypair RIVANJP-KEY
  exit
 !
 
 
 crypto pki authenticate RIVAN-JP
 
 
 crypto pki enroll RIVAN-JP
 
~~~

<br>


-----BEGIN CERTIFICATE REQUEST-----

-----END CERTIFICATE REQUEST-----


<br>


~~~
!@NetOps
openssl x509 \
  -req \
  -in utmjp.csr \
  -CA subca.crt \
  -CAkey subca.key \
  -CAcreateserial \
  -out utmjp.crt \
  -days 30 \
  -extensions v3_leaf_req \
  -extfile utmjp.cnf
~~~


<br>


~~~
!@UTM-JP
conf t
 crypto pki import RIVAN-JP certificate
 
~~~


<br>
<br>

---
&nbsp;


## Site to Site VPN (via RSA-SIG Authentication)

### STEP 1 - PHASE 1 (IKEv2)

~~~
!@UTM-PH
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption __-__-__
  integrity __
  group __
 !
 crypto ikev2 policy IKEV2-POL
  proposal IKEV2-PROP
 !
 ! crypto ikev2 keyring VPN-KEYRING
  ! peer RIVANJP-PEER
  !  address __.__.__.__
  !  pre-shared-key _________
 !
 crypto ikev2 profile IKEV2-PROF
  match identity remote address __.__.__.__  __.__.__.__
  authentication remote rsa-sig
  authentication local rsa-sig
  pki trustpoint RIVAN-PH
  ! keyring local VPN-KEYRING
   end
~~~


<br>


~~~
!@UTM-JP
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption __-__-__
  integrity __
  group __
 !
 crypto ikev2 policy IKEV2-POL
  proposal IKEV2-PROP
 !
 ! crypto ikev2 keyring VPN-KEYRING
  ! peer RIVANPH-PEER
  !  address __.__.__.__
  !  pre-shared-key _________
 !
 crypto ikev2 profile IKEV2-PROF
  match identity remote address __.__.__.__  __.__.__.__
  authentication remote rsa-sig
  authentication local rsa-sig
  pki trustpoint RIVAN-JP
  ! keyring local VPN-KEYRING
   end
~~~


<br>
<br>

---
&nbsp;


### STEP 2 - PHASE 2 (IPSEC)

~~~
!@UTM-PH, UTM-JP
conf t
 crypto ipsec transform-set TSET __-__  ___  __-__-__
  mode transport
 !
 crypto ipsec profile VPN-IPSEC-PROF
  set transform-set TSET
  set ikev2-profile IKEV2-PROF
  end
~~~


<br>
<br>

---
&nbsp;


### STEP 3 - TUNNEL PROPERTIES

~~~
!@UTM-PH
conf t
 int tun1
  ip add __.__.__.__  __.__.__.__
  tunnel source __
  tunnel destination __.__.__.__
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile VPN-IPSEC-PROF
  end
~~~

<br>

~~~
!@UTM-JP
conf t
 int tun1
  ip add __.__.__.__  __.__.__.__
  tunnel source __
  tunnel destination __.__.__.__
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile VPN-IPSEC-PROF
  end
~~~


<br>
<br>

---
&nbsp;


### STEP 4 - Remote Subnets / Interesting Traffic

~~~
!@UTM-PH
conf t
 ip route __.__.__.__   __.__.__.__   __.__.__.__
 ip route __.__.__.__   __.__.__.__   __.__.__.__
 end
~~~

<br>

~~~
!@UTM-JP
conf t
 ip route __.__.__.__   __.__.__.__   __.__.__.__
 end
~~~


<br>
<br>

---
&nbsp;


### Verification

!@VPN-PH, VPN-JP
show crypto isakmp sa
show crypto ipsec sa
