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

2. __VPN-JP__
- Name of Virtual Machine: VPN-JP
- Deployment Options: Small
- Bootstrap:
  - Router Name: VPN-PH
  - Login User: admin
  - Login Pass: pass

<br>

- Network Adapter: NAT
- Network Adapter 2: VMNet2
- Network Adapter 3: VMNet4

<br>
<br>

Deploy 2 YVM-v6
1. __BLDG-PH__
- Name of Virtual Machine: BLDG-PH
- Network Adapter: VMNet3

<br>

2. __BLDG-JP__
- Name of Virtual Machine: BLDG-JP
- Network Adapter: VMNet4

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
  ip add 192.168.102.11 255.255.255.0
  no shut
 int g3
  ip add 10.10.10.11 255.255.255.0
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
  ip add 192.168.102.12 255.255.255.0
  no shut
 int g3
  ip add 20.20.20.12 255.255.255.0
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
ifconfig eth0 10.10.10.10 netmask 255.255.255.0 up
route add default gw 10.10.10.11
ping 10.10.10.11
~~~

<br>

~~~
@BLDG-JP
sudo su
ifconfig eth0 20.20.20.20 netmask 255.255.255.0 up
route add default gw 20.20.20.12
ping 20.20.20.12
~~~

&nbsp;
---
&nbsp;

### Access WEB GUI
Open a browser and enter the Gig2 IP address of the VPN.  

- http://192.168.102.11/

- http://192.168.102.12/
