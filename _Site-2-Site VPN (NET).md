
<!-- Your Monitor Number == #$34T# -->

## Prerequisite
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

### Deployment
Note the following VM Files:
- CSR1000v 17.x = VPN-EDGE
- YVM-v6 = BLDG

<br>

Deploy 2 CSR1000v:
1. __VPN-PH__
- Name of Virtual Machine: VPN-PH
- Deployment Options: Small
- Bootstrap:
  - Router Name: VPN-PH
  - Login User: admin
  - Login Pass: pass
  - MGMT Interface: GigabitEthernet2
  - MGMT IP: 10.255.10.254/24
  - Feature - Telnet & SSH: Enabled

<br>

- Network Adapter: NAT
- Network Adapter 2: VMNet15
- Network Adapter 3: VMNet3

<br>

2. VPN-JP
- Name of Virtual Machine: VPN-JP
- Deployment Options: Small
- Bootstrap:
  - Router Name: VPN-PH
  - Login User: admin
  - Login Pass: pass
  - MGMT Interface: GigabitEthernet2
  - MGMT IP: 10.69.255.6/29
  - Feature - Telnet & SSH: Enabled

<br>

- Network Adapter: NAT
- Network Adapter 2: VMNet16
- Network Adapter 3: VMNet4

<br>
<br>

Deploy 2 YVM-v6
1. BLDG-PH
Name of Virtual Machine: BLDG-PH
- Network Adapter: VMNet3

<br>

2. BLDG-JP-1
Name of Virtual Machine: BLDG-JP-1
- Network Adapter: VMNet4

<br>

3. BLDG-JP-2
Name of Virtual Machine: BLDG-JP-2
- Network Adapter: VMNet4

<br>

Delpoy NetOps VM:
- Name: NetOps
- Network Adapter: NAT
- Network Adapter 2: Bridged (Replicate)
- Network Adapter 3: VMNet15
- Network Adapter 4: VMNet16

&nbsp;
---
&nbsp;

### Configuration
~~~
!@VPN-PH
conf t
 hostname VPN-PH
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
  ip add 10.255.10.254 255.255.255.0
  no shut
 int g3
  ip add 10.11.11.113 255.255.255.224
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 end
wr
~~~

<br>

~~~
!@VPN-JP
conf t
 hostname VPN-JP
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
  ip add 10.69.255.6 255.255.255.248
  no shut
 int g3
  ip add 10.21.21.213 255.255.255.240
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 end
wr
~~~

<br>

~~~
@BLDG-PH
sudo su
ifconfig eth0 10.11.11.126 netmask 255.255.255.224 up
route add default gw 10.11.11.113
ping 10.11.11.113
~~~

<br>

~~~
@BLDG-JP-1
sudo su
ifconfig eth0 10.21.21.209 netmask 255.255.255.240 up
route add default gw 10.21.21.213
ping 10.21.21.213
~~~

<br>

~~~
@BLDG-JP-2
sudo su
ifconfig eth0 10.21.21.210 netmask 255.255.255.240 up
route add default gw 10.21.21.213
ping 10.21.21.213
~~~

<br>

> [!IMPORTANT]
> Verify if the VM Interface matches the VMNet Attatchment
~~~
!@NetOps
nmcli connection add \
type ethernet \
con-name TunayNaLAN \
ifname ens192 \
ipv4.method manual \
ipv4.addresses 10.#$34T#.1.6/24 \
autoconnect yes
nmcli connection up TunayNaLAN

nmcli connection add \
type ethernet \
con-name VMNET15 \
ifname ens224 \
ipv4.method manual \
ipv4.addresses 10.255.10.6/24 \
autoconnect yes
nmcli connection up VMNET15

nmcli connection add \
type ethernet \
con-name VMNET16 \
ifname ens256 \
ipv4.method manual \
ipv4.addresses 10.69.255.4/29 \
autoconnect yes
nmcli connection up VMNET16
~~~

<br>

### Access WEB GUI
Open a browser and enter the Gig2 IP address of the VPN.  

- http://10.255.10.254/
- http://10.69.255.6/


<br>
<br>

---
&nbsp;


### Add User Accounts
~~~
!@All BLDG
sudo su
adduser admin
~~~

<br>

> New Password:    pass
> Retype Password: pass

<br>

| VPN-PH |     VARS       | BLDG-JP  |
| ---    | ---            | ---      |
|        | Authenticate   |          | 
|        |                |          |
|        | PHASE 1        |          |
|        | Encryption     |          |
|        | Integrity      |          |
|        | Group          |          |
|        |                |          |
|        | PHASE 2        |          |
|        | ESP-Encryption |          |
|        | ESP-Hash       |          |
|        | Tunnel IP      |          |
|        | Tunnel Mask    |          |
|        | Source Int     |          |
|        | Peer IP        |          |
|        | Remote Subnets |          |

<br>
<br>

---
&nbsp;

## Cryptography 
### RSA Key Pair
~~~
!@NetOps
ssh-keygen -t rsa -b 1024 -f rsa.key
fold -w 46 rsa.key.pub
~~~

<br>

or

<br>

~~~
!@NetOps
openssl genpkey -algorithm RSA -out rsa_priv.key -pkeyopt rsa_keygen_bits:1024
openssl pkey -in rsa_priv.key -pubout -out rsa_pub.key
~~~

<br>

Verify:
~~~
!@cmd
ssh -l root 10.m.1.6/24
~~~

<br>

~~~
The authenticity of host '10.91.1.6 (10.91.1.6)' can't be established.
ED25519 key fingerprint is SHA256:K1lJBH2lpRJ3UkDLhvobaAwCMwPaCcQuWB5bIb8heBg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
~~~

<br>

Where are the host private keys located on linux?
~~~
!@NetOps
cd /etc/ssh/
~~~

<br>

Verify the fingerprint
~~~
!@NetOps
ssh-keygen -lf ssh_host_ed25519_key.pub
~~~

<br>

Are the RSA keys used for payload encryption?

<br>
<br>

---
&nbsp;

## Key Exchange
### Diffie-Hellman 
~~~
!@NetOps
openssl dhparam -out dhgroup.pem 1024

openssl genpkey -paramfile dhgroup.pem -out ph_priv.key
openssl pkey -in ph_priv.key -pubout -out ph_pub.key

openssl genpkey -paramfile dhgroup.pem -out jp_priv.key
openssl pkey -in jp_priv.key -pubout -out jp_pub.key  

openssl pkeyutl -derive -inkey ph_priv.key -peerkey jp_pub.key -out ph-shared.secret
openssl pkeyutl -derive -inkey jp_priv.key -peerkey ph_pub.key -out jp-shared.secret
~~~

<br>

View Private key contents
~~~
!@NetOps
openssl pkey -in rsa_priv.key -text -noout
~~~

&nbsp;
---
&nbsp;

### Encryption types
1. Asymmetrical
 - Private Key
 - Public Key

2. Symmetrical
 - Shared Key/Session Key

<br>

SSH - Server key is used only to verify authenticity.   
RSA - Generate Key Pair   

&nbsp;
---
&nbsp;

### SHA - Hash data for authenticity, hashes encrypted data
~~~
!@NetOps
echo my_gcash_info > data.txt
sha256sum data.txt
~~~

<br>

or

<br>

~~~
!@cmd
certutil -hashfile data.txt md5
~~~

&nbsp;
---
&nbsp;

### AES - Encrypts data in Session
~~~
!@NetOps
openssl enc -aes-256-cbc -in data.txt -out data.enc
~~~

<br>

Salted - add bits of info to the encryption to prevent reverse eng
Make sure to produce different output per encryption.

&nbsp;
---
&nbsp;

### DH-GROUP - Establish parameters for ephemeral keys to keep key exchange secure. Shared secret (a lot of math)
~~~
!@NetOps
openssl dhparam -out dhgroup.pem 1024
~~~

&nbsp;
---
&nbsp;

### HMAC-SHA256 - authenticity and identity of payload, use the hash of the shared session key to be added to the unencrypted message, then together both are hashed.
~~~
!@NetOps
sha256sum ph-shared.secret
nano pre.hmac
~~~

<br>

~~~
!@NetOps
openssl enc -aes-256-cbc -in pre.hmac -out hmac
~~~

&nbsp;
---
&nbsp;

### Signature - Encrypts Hash of the message using Ephemeral Priv Key/longterm private key
~~~
!@NetOps
sha256sum data.txt > hash.file
openssl pkeyutl -encrypt -inkey ph_priv.key -in hash.file -out Signature
~~~

&nbsp;
---
&nbsp;

### Fingerprint - Hash of the shared secret/public key, separate from the message.
~~~
!@NetOps
sha256sum rsa_pub.key
~~~
