
![diag](<img/S2S-Blank.png>)

<br>
<br>

---
&nbsp;

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

CSR1000v:
  Name: UTM-PH
  
  | NetAdapter   |        |
  | ---          | ---    |
  | NetAdapter   | NAT    |
  | NetAdapter 2 | VMNet2 |
  | NetAdapter 3 | VMNet3 |
  

CSR1000v:
  Name: UTM-JP
  
  | NetAdapter   |        |
  | ---          | ---    |
  | NetAdapter   | NAT    |
  | NetAdapter 2 | VMNet2 |
  | NetAdapter 3 | VMNet4 |
  

TinyCore (yvm.ova):
  Name: BLDG-PH-1
  
  | NetAdapter   |                    |
  | ---          | ---                |
  | NetAdapter   | VMNet3             |


TinyCore (yvm.ova):
  Name: BLDG-JP-1
  
  | NetAdapter   |                    |
  | ---          | ---                |
  | NetAdapter   | VMNet4             |


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
!@BLDG-PH-1
sudo su
ifconfig eth0 11.11.11.100 netmask 255.255.255.224 up
route add default gw 11.11.11.113
ping 11.11.11.113
~~~

<br>

~~~
!@BLDG-JP-1
sudo su
ifconfig eth0 21.21.21.211 netmask 255.255.255.240 up
route add default gw 21.21.21.213
ping 21.21.21.213
~~~


&nbsp;
---
&nbsp;


### Access WEB GUI
Open a browser and enter the Gig2 IP address of the VPN.  

- http://192.168.102.11/

- http://192.168.102.12/


&nbsp;
---
&nbsp;


### Create Accounts in Linux

~~~
!@BLDH-PH, BLDG-JP
sudo su
adduser admin

> pass
> pass
~~~
