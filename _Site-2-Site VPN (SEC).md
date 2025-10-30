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

<br>

- Network Adapter: NAT
- Network Adapter 2: VMNet2
- Network Adapter 3: VMNet3

<br>

2. VPN-JP
- Name of Virtual Machine: VPN-JP
- Deployment Options: Small
- Bootstrap:
  - Router Name: VPN-JP
  - Login User: admin
  - Login Pass: pass

<br>

- Network Adapter: NAT
- Network Adapter 2: VMNet2
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
- Network Adapter 3: Host-only
- Network Adapter 4: Host-only

&nbsp;
---
&nbsp;













































_____________________
********************* SETUP VMNETS


VMWare > Edit > Virtual Network Editor

Add/Edit the following VMNets:

VMNet 2, 
	VMNet Info: Host-only
	IP address: 192.168.102.0
	Subnet Mask: 255.255.255.0
	DHCP: Unchecked

VMNet 3, 
	VMNet Info: Host-only
	IP address: 192.168.103.0
	Subnet Mask: 255.255.255.0
	DHCP: Unchecked

VMNet 4, 
	VMNet Info: Host-only
	IP address: 192.168.104.0
	Subnet Mask: 255.255.255.0
	DHCP: Unchecked
	
VMNet 8(NAT)
	VMNet Info: NAT
	IP address: 208.8.8.0
	Subnet Mask: 255.255.255.0
	DHCP: Checked
	
_____________________
********************* DEPLOY CSRS AND LINUX

Note the following VM Files:

CSR1000v 17.x = VPN-EDGE
YVM-v6 = BLDG


Deploy 2 CSR1000v

1. VPN-PH
	Name of Virtual Machine: VPN-PH
	Deployment Options: Small
	Bootstrap:
		Router Name: VPN-PH
		Login User: admin
		Login Pass: pass

	Network Adapter: NAT
	Network Adapter 2: VMNet2
	Network Adapter 3: VMNet3
		
2. VPN-JP
	Name of Virtual Machine: VPN-JP
	Deployment Options: Small
	Bootstrap:
		Router Name: VPN-JP
		Login User: admin
		Login Pass: pass

	Network Adapter: NAT
	Network Adapter 2: VMNet2
	Network Adapter 3: VMNet4

Deploy 2 YVM-v6

1. BLDG-PH
	Name of Virtual Machine: BLDG-PH

	Network Adapter: VMNet3
	
2. BLDG-JP-1
	Name of Virtual Machine: BLDG-JP-1
	
	Network Adapter: VMNet4

3. BLDG-JP-2
  Name of Virtual Machine: BLDG-JP-2

_____________________
********************* CONFIGURE DEVICES

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
  ip add 192.168.102.12 255.255.255.0
  no shut
 int g3
  ip add 21.21.21.213 255.255.255.240
  ip add 22.22.22.223 255.255.255.192 secondary
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 end
wr


!@BLDG-PH
sudo su
ifconfig eth0 11.11.11.100 netmask 255.255.255.224 up
route add default gw 11.11.11.113
ping 11.11.11.113

!@BLDG-JP-1
sudo su
ifconfig eth0 21.21.21.211 netmask 255.255.255.240 up
route add default gw 21.21.21.213
ping 21.21.21.213

!@BLDG-JP-2
sudo su
ifconfig eth0 22.22.22.221 netmask 255.255.255.192 up
route add default gw 22.22.22.223
ping 22.22.22.223

_____________________
********************* ACCESS VPN WEB GUI

Open a browser and enter the Gig2 IP address of the VPN.

http://192.168.102.11/

http://192.168.102.12/












<br>
<br>

---
&nbsp;

## Site-to-Site VPN (Signature) - ROOT CA
Deploy the following VMs:
- NetOps

| VM        | NetAdapter | NetAdapter 2       | NetAdapter 3 | NetAdapter 4 |
| ---       | ---        | ---                | ---          | ---          |
| VPN-PH    | NAT        | Bridge (Replicate) | Host-Only    | Host-Only    |

<br>

Login: root  
Pass: C1sc0123  

&nbsp;
---
&nbsp;

### 01. Identify the NAT IP & connect via SecureCRT
~~~
!@NetOps
ip -4 addr
~~~

&nbsp;
---
&nbsp;

### 02. Set a static IP address to connect to the LAN
~~~
!@NetOps
nmcli connection add type ethernet con-name TunayNaLAN \
ifname ens192 \
ipv4.method manual \
ipv4.addresses \
10.#$34T#.1.6/24 \
autoconnect yes

nmcli connection up TunayNaLAN
~~~

&nbsp;
---
&nbsp;

### 03. Set static routes
~~~
!@NetOps
ip route add 10.0.0.0/8 via 10.#$34T#.1.4
ip route add 200.0.0.0/24 via 10.#$34T#.1.4
~~~

<br>
<br>

---
&nbsp;

### Activity 02: Create a Selfsigned Certificate with the following subject names:
For the CA (NetOps)
- Country Name [XX]:                         PH
- State or Province Name []:                 NCR
- Locality Name [Default City]:              Makati
- Organization Name [Default Company Ltd]:   Rivancorp
- Organizational Unit Name (eg, section) []: HQ
- Common Name []:                            rivan.com
- Email Address []:                          admin@rivancorp.com
- Subject Alt Names:                         rivan.com  www.rivan.com  api.rivan.com  10.#$34T#.1.6

&nbsp;
---
&nbsp;

### 01. Create a directory for the certstore
~~~
!@NetOps
mkdir certstore
cd certstore
~~~

&nbsp;
---
&nbsp;

### 02. Create a Private key with a Selfsigned Certificate (SSH keys:ssh-keygen vs TLS/SSL keys:openssl)
~~~
!@NetOps
openssl req -x509 -newkey rsa:2048 -days 365 -keyout rivan.key -out ca-rivan.crt -nodes
~~~

<br>

Verify the certificate  

~~~
!@NetOps
openssl x509 -in ca-rivan.crt -text -noout
~~~

&nbsp;
---
&nbsp;

### 03. Create a configuration file to specify subject alternate names.
~~~
!@NetOps
nano ext.cnf
~~~

<br>

~~~
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no

[ req_distinguished_name ]
C  = PH
ST = NCR
L  = Makati
O  = Rivancorp
OU = HQ
CN = RivanCorporation

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1   = rivan.com
DNS.2   = www.rivan.com
DNS.3   = api.rivan.com
IP.1    = 10.#$34T#.1.6
~~~

&nbsp;
---
&nbsp;

### 04. Generate root CA with subject alt names
~~~
!@NetOps
openssl req -x509 -newkey rsa:2048 -days 365 -keyout rivan.key -out ca-rivan.crt -nodes -config ext.cnf -extensions v3_req
~~~

&nbsp;
---
&nbsp;

### 05. Import CA to Cisco Devices
~~~
!@VPN-PH
conf t
 crypto pki trustpoint rivantrust
  enrollment terminal pem
  hash sha512
  subject-name CN=siteph.rivan.com, C=PH, ST=NCR, L=Makati, O=Rivancorp, OU=SitePH, E=siteph@rivancorp.com
  subject-alt-name siteph.rivan.com
  subject-alt-name ph.rivan.com
  storage nvram: 
  primary
  revocation-check none
  rsakeypair rivankeys
  exit
 crypto pki authenticate rivantrust
> Paste the CA
~~~

<br>

~~~
!@VPN-JP
conf t
 crypto pki trustpoint rivantrust
  enrollment terminal pem
  hash sha512
  subject-name CN=siteph.rivan.com, C=JP, ST=Kanto, L=Tokyo, O=Rivancorp, OU=SiteJP, E=sitejp@rivancorp.com
  subject-alt-name sitejp.rivan.com
  subject-alt-name jp.rivan.com
  storage nvram: 
  primary
  revocation-check none
  rsakeypair rivankeys
  exit
 crypto pki authenticate rivantrust
> Paste the CA
~~~

&nbsp;
---
&nbsp;

### 06. Generate a CSR for both Routers
~~~
!@VPN-PH, VPN-JP
crypto pki enroll rivantrust

> Outputs a CSR . Must be signed by the CA
~~~

&nbsp;
---
&nbsp;

### 07. Import the CSR to the CA Server (NetOps)
~~~
!@NetOps
nano req-ph.pem
> paste VPN-PH's CSR
> ctrl + s (save)
> ctrl + x (exit)
~~~

<br>

~~~
!@NetOps
nano req-jp.pem
> paste VPN-JP's CSR
> ctrl + s (save)
> ctrl + x (exit
~~~

&nbsp;
---
&nbsp;

8. Create Configuration files for each VPN Routers
~~~
!@NetOps
nano vpnph.cnf
~~~

<br>

~~~
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no

[ req_distinguished_name ]
C  = PH
ST = NCR
L  = Makati
O  = Rivancorp
OU = SitePH
CN = RivanCorpPH

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1   = rivan.ph
DNS.2   = www.rivan.ph
DNS.3   = api.rivan.ph
IP.1    = 208.8.8.11
~~~

<br>

~~~
!@NetOps
nano vpnjp.cnf
###########

[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no

[ req_distinguished_name ]
C  = JP
ST = Kanto
L  = Tokyo
O  = Rivancorp
OU = SiteJP
CN = RivanCorpJP

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1   = rivan.jp
DNS.2   = www.rivan.jp
DNS.3   = api.rivan.jp
IP.1    = 208.8.8.12
~~~

&nbsp;
---
&nbsp;

### 08. Sign the CSRs
~~~
!@NetOps
openssl x509 -req -in req-ph.pem -CA ca-rivan.crt -CAkey rivan.key -out signed-ph.pem -extfile vpnph.cnf -extensions v3_req
openssl x509 -req -in req-jp.pem -CA ca-rivan.crt -CAkey rivan.key -out signed-jp.pem -extfile vpnjp.cnf -extensions v3_req
~~~

<br>

~~~
!@NetOps
cat signed-ph.pem
cat signed-jp.pem
~~~

&nbsp;
---
&nbsp;

### 09. Import the signed CSRs
~~~
!@VPN-PH, VPN-JP
conf t
 crypto pki import rivantrust certificate

> Paste the signed CSR
~~~

&nbsp;
---
&nbsp;

### Verify Cisco Certificates and Trustpoints
~~~
!@VPNs
show crypto pki certificates
show crypto pki trustpoint
~~~

<br>
<br>

---
&nbsp;

## Site-to-Site VPN (Sign)

### 01. GRE Tunnel
~~~
!@VPN-PH
conf t
 int tun1
  ip add 172.16.10.1 255.255.255.0
  tunnel source g1
  tunnel destination 208.8.8.12
  tunnel mode gre ip
  end
~~~

<br>

~~~
!@VPN-JP
conf t
 int tun1
  ip add 172.16.10.2 255.255.255.0
  tunnel source g1
  tunnel destination 208.8.8.11
  tunnel mode gre ip
  end
~~~

&nbsp;
---
&nbsp;

### 02. Routing Interesting traffic
~~~
!@VPN-PH
conf t
 ip route 21.21.21.208 255.255.255.240 172.16.10.2
 ip route 22.22.22.192 255.255.255.192 172.16.10.2
 end
~~~

<br>

~~~
!@VPN-JP
conf t
 ip route 11.11.11.96 255.255.255.224 172.16.10.1
 end
~~~

&nbsp;
---
&nbsp;

### 03. Configure ISAKMP Policy
~~~
!@VPN-PH, VPN-JP
conf t
 crypto isakmp policy 1
  authentication rsa-sig
  encryption aes 256
  hash sha512
  group 14
  end
~~~

&nbsp;
---
&nbsp;

### 04. Configure IPSec Profile
~~~
!@VPN-PH, VPN-JP
conf t
 crypto ipsec transform-set IPSECTUNNEL esp-aes 256 esp-sha-hmac
  mode transport
  exit
 crypto ipsec profile RIVAN
  set transform-set IPSECTUNNEL
  set pfs group14
  end
~~~

&nbsp;
---
&nbsp;

### 05. Apply IPSec Profile Protection to Tunnel
~~~
!@VPN-PH, VPN-JP
conf t
 int tunnel 1
  tunnel protection ipsec profile RIVAN 
  end
~~~

<br>

Verify:
~~~
!@VPN-PH, VPN-JP
show crypto isakmp sa
show crypto ipsec sa
~~~

<br>
<br>

---
&nbsp;
