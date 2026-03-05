S2S-VPN via Active Directory Certificate Services

## CERTIFICATE AUTHORITY

### Deployment

~~~
!@UTM-PH
conf t
 hostname UTM-PH
 enable secret pass
 service password-encryption
 no logging cons
 ip domain lookup
 ip domain lookup source-interface G2
 ip name-server 192.168.102.8
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
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
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
 ip domain lookup
 ip domain lookup source-interface G2
 ip name-server 192.168.102.8
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
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~


<br>


~~~
!@BLDG-PH
sudo su
ifconfig eth0 11.11.11.111 netmask 255.255.255.224 up
route add default gw 11.11.11.113
ping 11.11.11.113
~~~


<br>


~~~
!@BLDG-JP
sudo su
ifconfig eth0 21.21.21.211 netmask 255.255.255.240 up
route add default gw 21.21.21.213
ping 21.21.21.213
~~~


<br>


### STEP 1 - Setup WinServer

| NetAdapter  | VMNet  | IP Address    | 
| ---         | ---    | ---           |
| 1           | NAT    | 208.8.8.8     |
| 2           | VMNet2 | 192.168.102.8 |

<br>

~~~
!@Powershell
set-netfirewallprofile -name private,public,domain -enabled false
rename-computer ccnp69.com
ncpa.cpl
~~~


<br>


### STEP 2 - Install Active Directory Domains and Services
Create a Service Account:
- Active Directory and Users and Computers

| New User    |              |
| ---         | ---          |
| User Name   | ca           |
| Full Name   | CERTAUTH     |
| Password    | C1sc0123     |
| Pass Policy | Never Expire |
| Member of   | IIS_IUSRS    |


<br>


### STEP 3 - Install Active Directory Certificate Services

Afterwards, install ADCS Add-Ons:
- Certificate Enrollment Policy Web Service  
- Certificate Enrollment Web Service         
- Certificate Authority Web Enrollment
- Network Device Enrollment Service


<br>


### STEP 4 - Create DNS Mapping for both Device
- Reverse Lookup Zone : 208.8.8.0
- A Record : utmph.ccnp69.com : 208.8.8.11
- A Record : utmjp.ccnp69.com : 208.8.8.12


<br>


### STEP 5 - Configure Certificates on Cisco UTM-PH & UTM-JP

~~~
!@UTM-PH
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://192.168.102.8/certsrv/mscep/mscep.dll
  usage ike
  usage ssl-server
  usage ssl-client
  serial-number
  fqdn utmph.ccnp69.com
  ip-address 208.8.8.11
  subject-name CN=UTM-PH,OU=NOC,O=RIVANCORP,L=MAKATI,ST=NCR,C=PH
  subject-alt-name utmph.ccnp69.com
  revocation-check none
  source interface GigabitEthernet1
  rsakeypair CERTKEY
  end
~~~

<br>

~~~
!@UTM-JP
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://192.168.102.8/certsrv/mscep/mscep.dll
  usage ike
  usage ssl-server
  usage ssl-client
  serial-number
  fqdn utmjp.ccnp69.com
  ip-address 208.8.8.12
  subject-name CN=UTM-JP,OU=NOC,O=RIVANCORP,L=TOKYO,ST=KANTO,C=JP
  subject-alt-name utmjp.ccnp69.com
  revocation-check none
  source interface GigabitEthernet1
  rsakeypair CERTKEY
  end
~~~


<br>


### STEP 6 - Access CA Web Enrollment

Set Routes for trustpoints:

~~~
!@cmd
route add 208.8.8.11 mask 255.255.255.255 192.168.102.11
route add 208.8.8.12 mask 255.255.255.255 192.168.102.12
~~~


<br>


http://192.168.102.8/certsrv/mscep/mscep.dll


Grab the Hash & Challenge Password
- Hash: 7AE62A49 C4D0A3B3 938FE5C1 0BCFD0B0 
- Pass: DE005172D85A06D6 


<br>


### STEP 7 - Enroll Network Devices

~~~
!@UTM-PH,UTM-JP
conf t
 crypto pki enroll CCNPTRUST
~~~


<br>


## S2S-VPN

### STEP 1 - Phase 1 (IKEv2)

~~~
!@UTM-PH
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption aes-cbc-256
  integrity sha256
  group 14
 !
 crypto ikev2 policy IKEV2-POL
  proposal IKEV2-PROP
 !
 ! crypto ikev2 keyring VPN-KEYRING
  ! peer RIVANJP-PEER
  !  address 208.8.8.12
  !  pre-shared-key C1sc0123
 !
 crypto ikev2 profile IKEV2-PROF
  match identity remote address 208.8.8.12 255.255.255.255
  authentication remote rsa-sig
  authentication local rsa-sig
  pki trustpoint CCNPTRUST
  ! keyring local VPN-KEYRING
   end
~~~

<br>

~~~
!@UTM-JP
conf t
 crypto ikev2 proposal IKEV2-PROP
  encryption aes-cbc-256
  integrity sha256
  group 14
 !
 crypto ikev2 policy IKEV2-POL
  proposal IKEV2-PROP
 !
 ! crypto ikev2 keyring VPN-KEYRING
  ! peer RIVANPH-PEER
  !  address 208.8.8.11
  !  pre-shared-key C1sc0123
 !
 crypto ikev2 profile IKEV2-PROF
  match identity remote address 208.8.8.11 255.255.255.255
  authentication remote rsa-sig
  authentication local rsa-sig
  pki trustpoint CCNPTRUST
  ! keyring local VPN-KEYRING
   end
~~~


<br>


### STEP 2 - Phase 2 (IPSEC)

~~~
!@UTM-PH, UTM-JP
conf t
 crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
  mode transport
 !
 crypto ipsec profile VPN-IPSEC-PROF
  set transform-set TSET
  set ikev2-profile IKEV2-PROF
  end
~~~


### STEP 3 - Tunnel Properties

~~~
!@UTM-PH
conf t
 int tun1
  ip add 172.16.1.1 255.255.255.252
  tunnel source g1
  tunnel destination 208.8.8.12
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile VPN-IPSEC-PROF
  end
~~~

<br>

~~~
!@UTM-JP
conf t
 int tun1
  ip add 172.16.1.2 255.255.255.252
  tunnel source g1
  tunnel destination 208.8.8.11
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile VPN-IPSEC-PROF
  end
~~~


<br>


### STEP 4 - Remote Subnets / Interesting Traffic
~~~
!@UTM-PH
conf t
 ip route 21.21.21.208  255.255.255.248  172.16.1.2
 end
~~~

<br>

~~~
!@UTM-JP
conf t
 ip route 11.11.11.96  255.255.255.224  172.16.1.1
 end
~~~
