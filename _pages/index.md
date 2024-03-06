### 1. **ReInvent 2023 Networking**

AWS 2023년 한 해 확장성/용량/성능 개선 사항 발표

* **지속적인 리전 확장**
	1.  2023년에 2개 리전 추가, 이후 4개 리전 추가 예정
	2.  20개의 Direct Connect locations 추가
<br/>
* **CloudFront PoP 확장**
	![[Pasted image 20240306132413 1.png]]
	CloudFront는 전 세계 200개 이상의 도시에 600개 이상의 임베디드 POP를 배포하고 있음
	 네트워크 내 여러 위치에 신속하게 배포하여,
	 ==최종 사용자에게 더 가까운 곳==에 추가 용량을 구축할 수 있다.
<br/>
* **SRD 전송 기술 (Scalable Reliable Datagram)**
	기존 TCP의 Single Flow는 패킷 손실 없이 전송하기 위해 하나의 네트워크 경로로 모든 패킷을 보낸다.
	 ==> 이용 경로에 장애나 병목 현상이 발생하게 되면 지연시간이 증가하고 성능이 저하된다.
	 
	![[Pasted image 20240306132814 1.png]]
	SRD는 **여러 경로로 패킷을 쪼개 전송**하기 때문에, 이용하던 경로에 장애가 있다면 마이크로초 속도로 재송출을 하게 만들어졌다.

	![[Pasted image 20240306132915 1.png]]
	AWS 콘솔에서 특정 인스턴스 타입을 사용하고있는 Network Interface에서 **추가비용없이** 제공된다.

<br/>

### 2. VPN + Route53

#### **Site-to-Site VPN**

: **On-premise의 데이터 센터 혹은 지점과 AWS 클라우드 리소스 사이의 안전한 네트워크 연결 생성**
가상 프라이빗 게이트웨이 or Transit Gateway와 고객 게이트웨이 사이에 ==두개의 VPN 터널==을 제공

* **Site-to-Site VPN 구성 요소**
	1. 가상 프라이빗 게이트 웨이 혹은 Transit Gateway
		: AWS 환경을 VPN으로 연결 할 Endpoint
		
	2. 고객 게이트웨이 디바이스
		: Site-to-Site VPN 연결을 위해 고객 측에 설치된 물리적 디바이스 또는 소프트웨어 어플리케이션
		
	3. 고객 게이트웨이
		: 온프레미스 네트워크의 고객 게이트웨이 디바이스를 나타내는 AWS에서 생성하는 리소스
<br/>
* **Site-to-Site VPN 구성하기**

> [!NOTE] 시나리오
> AnyCompany는 현재 서울에 본사(headquater, HQ)를 두고있으며 On-prem IDC 또한 서울에 위치하여 사용하고 있습니다. 
> 1) 클라우드 워크로드의 도입으로 인하여 AWS로의 마이그레이션이 이루어지고 있지만 여전히 IDC의 몇몇 데이터 및 워크로드가 필요합니다. 
> 2) 또한, 브라질 상파울루 오피스와의 안정적인 네트워크 연결이 필요하나, 물리적으로 매우 먼 거리와 현지 공인망의 불안정성으로 인하여 네트워크 상황이 불안정합니다. 

---

![[Pasted image 20240306133807 1.png]]

서울 리전에 위 아키텍처와 같이 AWS VPC에 Transit Gateway (이하 TGW)가 배포되어있고,
온프레미스 IDC를 시뮬레이션한 환경에 Customer Gateway (이하 CGW)가 구성이 되어있는 상태

두 네트워크 사이에 Site-to-Site VPN을 구성한다.
<br/>

1. **Transit Gateway Attachment 생성하기**
	서울 리전에 TGW는 생성이 되어있으며, WPC와의 attachment도 설정이 되어있는 상황이다
	온프레미스 IDC와 TGW의 attachment를 생성해야한다.

<br/>
VPC 콘솔의 VPC -> Transit Gateway -> Transit Gateway attachments 메뉴에서 Create transit gateway attachment를 클릭한다.

![[Pasted image 20240306134214 1.png]]
Attachment type을 VPN으로 선택 후, TGW ID의 드롭다운 메뉴를 선택하여 생성되어있는 TGW를 선택한다. 
VPN Attachment에서 새로운 CGW와의 연결을 구성할 것이기 때문에 New를 선택 후 온프레미스 환경의 CGW 공인 IP를 입력해준다.

![[Pasted image 20240306134231 1.png]]

터널 옵션에 값을 입력 후 생성이 완료되면, Transit Gateway -> Transit Gateway Route Table로 이동,
Transit Gateway의 Default 라우팅 테이블을 선택 후 하단의 Association 탭에서 VPN이 associated 상태인지 확인한다.

![[Pasted image 20240306134347 1.png]]
<br/>

2. **VPN Connection 확인하기**
	TGW Attachment를 생성하면서  Attachment Type이 VPN이었기 때문에,
	이 과정에서 ==CGW와 VPN 연결이 함께 생성==된다. 
	
	생성된 VPN 연결을 확인하는 과정이다.

<br/>
VPN 콘솔의 VPN -> Site-to-Site VPN connections 메뉴를 클릭 후,
해당 터널을 선택하고 Tunnel Detail에서 터널의 IP 정보를 확인한다.

![[Pasted image 20240306134611 1.png]]
VPN 연결의 **가용성**을 위해 ==두 개의 터널을 모두 사용하는 것을 권장==한다. 
Outside IP 주소는 온프레미스 CGW 디바이스에서 IPsec 설정에 필요하므로 메모해둔다.

터널 정보 상단의 Downloads configuration 버튼을 클릭하면,
각 ==CGW 디바이스 제공 업체 별 configuration 파일==을 다운 받을 수 있다.

![[Pasted image 20240306134755 1.png]]
<br/>

3. **온프레미스 라우터 설정하기**
	VPN 터널을 확인 했을 때 down 상태였는데, 이를 활성화하기 위해선
	==고객 게이트웨이 디바이스 혹은 라우터에서 VPN 설정==을 해줘야한다.
	
	다이나믹 라우팅이 가능한 라우터 구성을 위해 사용된 라우터는 다음과 같다.
	- StrongSWAN (IPsec)
	- Quagga (BGP routing)

먼저, 온프레미스 고객 게이트웨이 디바이스 서버에 접속을 해준다.
그 후 아래 스크립트를 서버에서 생성 후, 실행 시켜 준다.

```
#!/bin/bash
#
#   StrongSwan(IPSec) + Quaaga (BGP) Setup Script for EC2 Virtual Router
#   - 2023/02/23
#
#  How to use
#    1. Allocate EIP to EC2 located at public subnet
#    2. Create CGW which is using EIP of EC2 virtual router, 
#       and then make S2S VPN tunnel connection between TGW and the CGW
#    3. From the VPN tunnel config, get two of Outside IPs and run this script
#

# CGW(EC2 Instance) Eth0
pCgwEth0Ip=$(hostname -i)
pCgwEip=$(curl -s ifconfig.me)
pCgwCidr="`echo $pCgwEth0Ip | cut -d "." -f 1-2`.0.0/16"

# IPSec Tunnel #1 Info
pTu1CgwOutsideIp=$pCgwEip
pTu1CgwInsideIp=169.254.11.2
pTu1VgwInsideIp=169.254.11.1

# IPSec Tunnel #2 Info
pTu2CgwOutsideIp=$pCgwEip
pTu2CgwInsideIp=169.254.12.2
pTu2VgwInsideIp=169.254.12.1

#BGP ASN and PSK Info
pVgwAsn=64512
pCgwAsn=65016
pTuPsk=strongswan_awsvpn

echo "=============================================================="
echo " Let's begin to set up IPSEC/BGP using StrongSWAN and Quagga  "
echo "--------------------------------------------------------------"
echo "  0. strongswan and quagga has been installed " 
echo "----------------------------------------------------------"
echo "  1. IPSec Info - Input VPN Tunnel Outside IP addresses "
read -p "    - Tunnel #1 Outside IP Addr : " pTu1VgwOutsideIp
read -p "    - Tunnel #2 Outside IP Addr : " pTu2VgwOutsideIp
echo "----------------------------------------------------------"
echo "  2. BGP Info -  ASN numbers are set as below"
echo "    - TGW ASN Number (64512-65534) : $pVgwAsn " 
echo "    - CGW ASN Number (64512-65534) : $pCgwAsn " 
echo "=========================================================="
read -p "  informations above is correct? If yes, please continue (y/N)? " answer2
echo

if [ "${answer2,,}" != "y" ]
then
    exit 100
fi

echo "3. Set IPSEC config on /etc/strongswan/ipsec.conf "

cat <<EOF > /etc/strongswan/ipsec.conf
#
# /etc/strongswan/ipsec.conf
#
conn %default
        # Authentication Method : Pre-Shared Key
        leftauth=psk
        rightauth=psk
        # Encryption Algorithm : aes-128-cbc
        # Authentication Algorithm : sha1
        # Perfect Forward Secrecy : Diffie-Hellman Group 2
        ike=aes128-sha1-modp1024!
        # Lifetime : 28800 seconds
        ikelifetime=28800s
        # Phase 1 Negotiation Mode : main
        aggressive=no
        # Protocol : esp
        # Encryption Algorithm : aes-128-cbc
        # Authentication Algorithm : hmac-sha1-96
        # Perfect Forward Secrecy : Diffie-Hellman Group 2
        esp=aes128-sha1-modp1024!
        # Lifetime : 3600 seconds
        lifetime=3600s
        # Mode : tunnel
        type=tunnel
        # DPD Interval : 10
        dpddelay=10s
        # DPD Retries : 3
        dpdtimeout=30s
        # Tuning Parameters for AWS Virtual Private Gateway:
        keyexchange=ikev1
        rekey=yes
        reauth=no
        dpdaction=restart
        closeaction=restart
        leftsubnet=0.0.0.0/0,::/0
        rightsubnet=0.0.0.0/0,::/0
        leftupdown=/etc/strongswan/ipsec-vti.sh
        installpolicy=yes
        compress=no
        mobike=no
conn TU1
        # Customer Gateway
        left=${pCgwEth0Ip}
        leftid=${pTu1CgwOutsideIp}
        # Virtual Private Gateway
        right=${pTu1VgwOutsideIp}
        rightid=${pTu1VgwOutsideIp}
        auto=start
        mark=100
conn TU2
        # Customer Gateway
        left=${pCgwEth0Ip}
        leftid=${pTu2CgwOutsideIp}
        # Virtual Private Gateway
        right=${pTu2VgwOutsideIp}
        rightid=${pTu2VgwOutsideIp}
        auto=start
        mark=200
EOF

echo "4. Set IPSEC config on /etc/strongswan/ipsec.secrets "

cat <<EOF > /etc/strongswan/ipsec.secrets
#
# /etc/strongswan/ipsec.secrets
#
${pTu1CgwOutsideIp} ${pTu1VgwOutsideIp} : PSK ${pTuPsk}
${pTu2CgwOutsideIp} ${pTu2VgwOutsideIp} : PSK ${pTuPsk}
EOF

echo "5. Set IPSEC tunnel options on /etc/strongswan/ipsec-vti.sh "

cat <<EOF > /etc/strongswan/ipsec-vti.sh
#!/bin/bash

#
# /etc/strongswan/ipsec-vti.sh
#

IP=\$(which ip)
IPTABLES=\$(which iptables)

PLUTO_MARK_OUT_ARR=(\${PLUTO_MARK_OUT//// })
PLUTO_MARK_IN_ARR=(\${PLUTO_MARK_IN//// })

case "\$PLUTO_CONNECTION" in
        TU1)
        VTI_INTERFACE=vti1
        VTI_LOCALADDR=${pTu1CgwInsideIp}/30
        VTI_REMOTEADDR=${pTu1VgwInsideIp}/30
        ;;
        TU2)
        VTI_INTERFACE=vti2
        VTI_LOCALADDR=${pTu2CgwInsideIp}/30
        VTI_REMOTEADDR=${pTu2VgwInsideIp}/30
        ;;
esac

case "\${PLUTO_VERB}" in
        up-client)
        #\$IP tunnel add \${VTI_INTERFACE} mode vti local \${PLUTO_ME} remote \${PLUTO_PEER} okey \${PLUTO_MARK_OUT_ARR[0]} ikey \${PLUTO_MARK_IN_ARR[0]}
        \$IP link add \${VTI_INTERFACE} type vti local \${PLUTO_ME} remote \${PLUTO_PEER} okey \${PLUTO_MARK_OUT_ARR[0]} ikey \${PLUTO_MARK_IN_ARR[0]}
        sysctl -w net.ipv4.conf.\${VTI_INTERFACE}.disable_policy=1
        sysctl -w net.ipv4.conf.\${VTI_INTERFACE}.rp_filter=2 || sysctl -w net.ipv4.conf.\${VTI_INTERFACE}.rp_filter=0
        \$IP addr add \${VTI_LOCALADDR} remote \${VTI_REMOTEADDR} dev \${VTI_INTERFACE}
        \$IP link set \${VTI_INTERFACE} up mtu 1436
        \$IPTABLES -t mangle -I FORWARD -o \${VTI_INTERFACE} -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
        \$IPTABLES -t mangle -I INPUT -p esp -s \${PLUTO_PEER} -d \${PLUTO_ME} -j MARK --set-xmark \${PLUTO_MARK_IN}
        \$IP route flush table 220
        #/etc/init.d/bgpd reload || /etc/init.d/quagga force-reload bgpd
        ;;
        down-client)
        #\$IP tunnel del \${VTI_INTERFACE}
        \$IP link del \${VTI_INTERFACE}
        \$IPTABLES -t mangle -D FORWARD -o \${VTI_INTERFACE} -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
        \$IPTABLES -t mangle -D INPUT -p esp -s \${PLUTO_PEER} -d \${PLUTO_ME} -j MARK --set-xmark \${PLUTO_MARK_IN}
        ;;
esac

# Enable IPv4 forwarding
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv4.conf.eth0.disable_xfrm=1
sysctl -w net.ipv4.conf.eth0.disable_policy=1
# Disable IPv4 ICMP Redirect
sysctl -w net.ipv4.conf.eth0.accept_redirects=0
sysctl -w net.ipv4.conf.eth0.send_redirects=0
EOF

sudo chmod +x /etc/strongswan/ipsec-vti.sh

echo "6. Set BGP setting using Quagga /etc/quagga/bgpd.conf "

cat <<EOF > /etc/quagga/bgpd.conf
#
# /etc/quagga/bgpd.conf
#
router bgp ${pCgwAsn}
bgp router-id ${pTu1CgwInsideIp}
neighbor ${pTu1VgwInsideIp} remote-as ${pVgwAsn}
neighbor ${pTu2VgwInsideIp} remote-as ${pVgwAsn}
network ${pCgwCidr}
EOF

echo "7. Start StrongSWAN and Quagga BGP "

sudo systemctl enable --now strongswan

sudo systemctl start zebra
sudo systemctl enable zebra
sudo systemctl start bgpd
sudo systemctl enable bgpd
sudo chmod -R 777 /etc/quagga/


sudo strongswan restart

echo "=========================================================="
echo " Using below command, verify IPSec Tunnel and BGP Routing tables"
echo " -. IPsec status    : sudo strongswan statusall  "
echo " -. Routing tables : sudo ip route  "
echo " -. BGP detail config : Enter teminal mode > sudo vtysh and then > show ip bgp "
echo "=========================================================="
```

해당 스크립트는 **아래의 과정을 자동으로 수행**한다.
* VPN Tunnel Outside IP를 입력 받는다.
* StrongSWAN을 이용한 IPsec 설정에 필요한 ipsec.conf, ipsec.secrets, ipsec-vti.sh를 /etc/strongswan에 생성한다.
* Quagga BGP를 설정한다.
* StrongSWAN과 Quagga를 시작한다.

실행이 완료되면 VPN 연결에서 확인한 터널의 Outside IP를 차례대로 입력하고 설정을 시작한다.

![[Pasted image 20240306140034 1.png]]

명령어를 사용하여 설정이 잘 적용되어있는지 확인한다.

`sudo strongswan statusall`

스크립트에서 입력한 Outside IP와 IPsec Connection을 잘 맺고있는지 확인 할 수 있다.
콘솔로 돌아와 VPN 터널의 Status가 모두 up인지 확인한다.

![[Pasted image 20240306140140 1.png]]

Site-to-Site VPN을 구성함으로써, AWS VPC와 온프레미스 IDC 간 네트워크 연결이 구성되어
==AWS 서버에서 IDC의 데이터 및 워크로드를 안전하게 사용==할 수 있게 되었다. 
</br>
#### **Route53 Resolver**
: 온프레미스 DNS를 Route53 DNS와 통합할 수 있는 ==하이브리드 DNS 인프라==를 생성할 수 있다.
R53 Resolver는 하이브리드 DNS 아키텍처를 **활성화하기 위한 도구를 제공**한다.

* **하이브리드 DNS 인프라를 생성하기 위한 구성 요소**
	1. Outbound Endpoint
		: VPC에서 온프레미스 네트워크나 다른 VPC로 DNS 쿼리를 보낼 수 있게한다.

	2. Inbound Endpoint
		: 온프레미스 네트워크나 다른 VPC에서 DNS 쿼리를 보낼 수 있게 한다.
	
	3. Route 53 Resolver Rule
		: 특정 DNS 도메인에 대한 DNS 쿼리를 온프레미스 DNS 서버로 전달하기 위한 규칙
	

> [!NOTE] 시나리오
> Anycompany는 On-Premises사용자들을 위해 DNS서버(Domain : anycompany.corp)가 구성되어 있습니다.
> AWS에는 Route53을 이용하여 DNS(awsanycompany.corp)이 구성되어 있습니다.
> 
>1) On-Premises 및 AWS 시스템간의 도메인을 통한 접속을 하고자 합니다.2) AWS시스템에 접속할 때, IPv4주소를 기억하지 않고 Domain(awsanycompany.corp)정보를 이용하여 접속하고자 합니다.
>2) AWS시스템에 접속할 때, IPv4주소를 기억하지 않고 Domain(awsanycompany.corp)정보를 이용하여 접속하고자 합니다.

---
기본 구성 상태는 다음과 같다.

![[Pasted image 20240306141339 1.png]]

| **온프레미스** | anycompany.corp | 192.168.2.250 (DNS) | 10.0.10.10 (WEB01) |
| ---- | ---- | ---- | ---- |
| **AWS** | **awscompany.corp** | **192.168.2.20 (WEB)** | **10.0.20.10 (WEB02)** |
</br>

1. **온프레미스와 IDC의 DNS 설정 확인**

IDC 서버에 접속해서 Server에 할당된 DNS 정보를 확인 한다.
```
cat /etc/resolv.conf

sh-4.2$ cat /etc/resolv.conf
; generated by /usr/sbin/dhclient-script
search anycompany.corp
options timeout:2 attempts:5
nameserver 192.168.2.250
```

DNS 서버에서 온프레미스 DNS의 **조건부 전달 규칙을 확인**한다.
AWS 서버 도메인에 대해, Endpoint로 전달하는 규칙을 확인 할 수 있다.
```
sudo su -
cat /etc/bind/named.conf.options

root@DNSSRV:~# cat /etc/bind/named.conf.options
options {
directory "/var/cache/bind";
recursion yes;
allow-query { any; };
forwarders {
8.8.8.8;
};
forward only;
auth-nxdomain no;
};
zone "awsanycompany.corp" {
type forward;
forward only;
forwarders { 10.0.1.250; 10.0.2.250; };
};
zone "ap-northeast-2.compute.internal" {
type forward;
forward only;
forwarders { 10.0.1.250; 10.0.2.250; };
};
```

<br/>

2. **AWS의 DNS 설정 확인**

Route43 -> Hosted Zone에서 Resource Record 정보를 확인한다.

**![[Pasted image 20240306141818 1.png]]**

AWS WEB 서버에 접속해 Resolver의 설정을 확인한다.
```
cat /etc/resolv.conf

sh-4.2$ cat /etc/resolv.conf
; generated by /usr/sbin/dhclient-script
search ap-northeast-2.compute.internal
options timeout:2 attempts:5
nameserver 10.0.0.2
```

Private Hosted Zone에 등록되어있는 RRset의 값이 정상적으로 확인되는지 아래와 같이 확인한다.
```
nslookup web1.awsanycompany.corp
nslookup web2.awsanycompany.corp

sh-4.2$ nslookup web1.awsanycompany.corp
Server: 10.0.0.2
Address: 10.0.0.2#53

Non-authoritative answer:
Name: web1.awsanycompany.corp
Address: 10.0.10.10

sh-4.2$ nslookup web2.awsanycompany.corp
Server: 10.0.0.2
Address: 10.0.0.2#53

Non-authoritative answer:
Name: web2.awsanycompany.corp
Address: 10.0.20.10
```
<br/>

3. **엔드포인트 구성하기**

Route53 콘솔 -> Resolver -> VPC를 선택 후 Resolver 구성을 위해 Configure Endpoint를 선택한다.

![[Pasted image 20240306142013 1.png]]

인바운드와 아웃바운드를 동시에 설정을 선택한다.

![[Pasted image 20240306142039 1.png]]

인바운드와 아웃바운드 엔드포인트를 구성한다.
가용성을 위해 ==최소 2개 이상의 가용 영역==에 Endpoint를 생성한다.

![[Pasted image 20240306142825 1.png]]
![[Pasted image 20240306142832 1.png]]

조건부 전달 규칙을 설정한다.

![[Pasted image 20240306142926 1.png]]

설정이 완료되면 아래와 같은 형태로 ==2개의 AZ에 인바운드/아웃바운드 엔드포인트==가 생성된다.

![[Pasted image 20240306143054 1.png]]
<br/>

4. **Hybrid DNS 구성 검증**

	1) Outbound Endpoint 구성 검증
		![[Pasted image 20240306143153 1.png]]
		* WEB 인스턴스에서 websrv.anycompany.corp 도메인에 대해 질의를 하면 R53으로 전송이 된다.
		* DNS 쿼리를 수신한 R53은 조건부 전송 규칙에 따라 도메인에 대해 192.168.2.250으로 전송한다.
		* DNS 쿼리는 **아웃바운드 엔드포인트를 통해 TGW로 전송**이 된다.
		* TGW는 목적지 주소를 확인하고 Site-to-Site VPN을 통해 온프레미스로 전송한다. 이를 수신한 CGW는 IDC내의 DNS 서버로 쿼리를 전달한다.
		* 온프레미스 IDC DNS는 수신된 쿼리의 내용을 확인하여 Websrv를 회신한다.
	<br/>
	2) Inbound Endpoint 구성 검증
		![[Pasted image 20240306143500 1.png]]
		* 온프레미스 Websrv 서버에서 web1.awsanycompany.corp 도메인에 대해 질의를 하면 DNS 서버로 전송이 된다.
		* DNS 쿼리를 수신한 DNS 서버는 조건부 전송 규칙에 따라 도메인에 대해 10.0.10.250 또는 10.0.20.250으로 전송한다.
		* CGW는 목적지 주소를 기반으로 Site-to-Site VPN을 통해 TGW로 쿼리 패킷을 전송한다.
		* TGW는 수신한 쿼리 정보를 확인하여 **인바운드 엔드포인트**로 전송한다.
		* 인바운드 엔드포인트는 이를 R53 Resolver에 전송하게 되며, R53은 Private Hosted Zone에서 Web 인스턴스의 IPv4 주소를 확인하여 회신한다. 

<br/>

#### **Client VPN**
: AWS 리소스 및 온프레미스 네트워크의 리소스에 안전하게 액세스 할 수 있도록 하는 
**관리형 클라이언트 기반 VPN 서비스**

AWS Client VPN은 완전 관리형의 탄력적 VPN 서비스로, 사용자 요구 사항에 맞추어 
==자동으로 확장하거나 축소==가 가능하다.

운영자 측면에서 기존의 설치형 VPN에 비해 서버/운영 관리 및 확장, 연단위의 라이센스 비용 측면에서 이점을 기대할 수 있다. 

> [!NOTE] 시나리오
> AnyCompany는 현재 서울에 본사(headquater, HQ)를 두고있으며 On-prem IDC 또한 서울에 위치하여 사용하고 있습니다. 
> 
> 3) 최근 팬데믹 시기를 지나며, 원격 및 재택 근무자가 늘어나 IDC로의 안전한 접근 뿐 아니라, AWS 워크로드로의 안전한 접속이 필요합니다.

원격 및 재택 근무자들에게 AWS 환경 및 이와 연결 된 온프레미스 네트워크에도 안전하게 접속할 수 있는 인프라를 제공해야 한다.

![[Pasted image 20240306144514 1.png]]
다음과 같이 ==클라이언트 엔드포인트를 생성하고 ENI를 통해 서브넷==에 접근한다. 
<br/>

1. **AWS Client VPN 클라이언트 인증**
	원격 접속자의 접근을 제어하기 위해 **인증을 사용**해야한다.
	
	인증이 성공하면 클라이언트가 Client VPN 엔드포인트에 연결하고, VPN 세션을 설정한다.
	실패하면 연결이 거부되고, 클라이언트가 VPN 세션을 연결할 수 있다.

![[Pasted image 20240306144723 1.png]]
**AWS Certificate Manager**에 등록된 인증서를 이용하여 ==상호 인증==하는 방식으로 인증을 구성한다.

OpenVPN easy-rsa를 사용하여 서버 및 클라이언트 인증서와 키를 생성 후, 
서버 인증서와 키를 ACM에 업로드 한다.

상호 인증을 하기 위해 다음 문서를 참고하여 OS에 맞게 인증서와 키를 생성한다.
https://docs.aws.amazon.com/ko_kr/vpn/latest/clientvpn-admin/mutual.html

인증서가 생성되었으면, AWS Certificate Manager 콘솔에서 인증서를 직접 업로드하면 된다.
인증서는 **서버와 클라이언트 두 개 다 업로드**를 해야한다.
<br/>

**서버**

- Certificate body : server.crt 파일 하단의 -----BEGIN CERTIFICATE----- 부터 -----END CERTIFICATE-----
- Certificate private key : server.key 파일의 -----BEGIN PRIVATE KEY----- 부터 -----END PRIVATE KEY-----
- Certificate chain : ca.crt 파일의 -----BEGIN CERTIFICATE----- 부터 -----END CERTIFICATE-----

**클라이언트**

- Certificate body : client1.domain.tld.crt 파일 하단의 -----BEGIN CERTIFICATE----- 부터 -----END CERTIFICATE-----
- Certificate private key : client1.domain.tld.key 파일의 -----BEGIN PRIVATE KEY----- 부터 -----END PRIVATE KEY-----
- Certificate chain : ca.crt 파일의 -----BEGIN CERTIFICATE----- 부터 -----END CERTIFICATE-----
<br/>

2. **Client VPN Endpoint 생성 및 설정**
	원격 접속자는 아래와 같이 Client VPN 엔드포인트에 접속해 VPN 세션을 설정하여 사용할 수 있다. 
	원격 접속을 허용하고자 하는 ==VPC에 서브넷을 따로 구성==하여 Endpoint와 연결하는 것이 모범사례다.
	
	
	![[Pasted image 20240306145058 1.png]]

Client VPN 엔드포인트를 생성해보자

VPC 콘솔의 VPN -> Virtual private network (VPN) -> Client VPN endpoints 메뉴에서 
VPN 엔드포인트를 생성하고 다음과 같이 입력한다.

![[Pasted image 20240306145247 1.png]]
![[Pasted image 20240306145253 1.png]]

Server certificate ARN 메뉴를 드롭다운하여 서버 인증서를 선택한다.

![[Pasted image 20240306145442 1.png]]

상호 인증을 위해 use mutual authentication을 선택하고 **클라이언트 인증서**를 선택한다.

![[Pasted image 20240306145506 1.png]]

Enable split-tunnel을 On 하지 않으면 모든 트래픽이 VPN Endpoint를 통하기 때문에,
**==로컬 PC의 인터넷이 되지 않는다==**.

![[Pasted image 20240306145604 1.png]]

해당 VPC의 **어떤 subnet에 Endpoint와 연결 될 ENI를 생성**할 지 선택해야한다.
Target network associations 탭에서 Client VPN 연결을 위한 서브넷을 선택하여 
a 서브넷과 b 서브넷을 둘 다 연결해준다.

![[Pasted image 20240306145758 1.png]]
![[Pasted image 20240306145810 1.png]]

위 과정을 완료하면 Client VPN Endpoint가 생성되고,
생성한 Endpoint에 대한 VPC 연결이 완료된다.
<br/>

3. **클라이언트 인증서 구성**
	생성한 Client Endpoint에 접속하기 위해선, **관리자가 배포한 인증서와 접속 프로그램이 필요**하다.

	접속 프로그램은 AWS Client VPN을 사용하며, 아래 링크에서 운영체제에 맞는 프로그램 다운이 가능하다.
	https://aws.amazon.com/ko/vpn/client-vpn-download/
	

먼저, 클라이언트 인증서 구성을 위해 생성된 Client Endpoint를 선택 한 뒤, Download Client Configuration 버튼을 선택한다.

![[Pasted image 20240306150255 1.png]]

팝업에서 Download Client Configuration 버튼을 선택에서 파일을 다운로드 받는다.
![[Pasted image 20240306150318 1.png]]

다운받은 downloaded-client-config.ovpn 파일을 편집기로 열어 **인증서와 키를 차례대로 추가**한다.
- <cert></cert> 사이에 client1.domain.tld.crt 파일의 -----BEGIN CERTIFICATE----- 부터 -----END CERTIFICATE-----
- <key></key> 사이에 client1.domain.tld.key 파일의 -----BEGIN PRIVATE KEY----- 부터 -----END PRIVATE KEY-----
- 마지막 라인에 verify-x509-name server name 는 삭제

파일을 저장 후 Client VPN 관리자는 이 인증서를 ==접속하고자 하는 원격 유저들==에게 배포한다.
<br/>

4. **AWS VPN Client 설치 및 설정**

앞에서 설치한 VPN Client를 실행 후 File -> Manage Profiles 메뉴로 들어간다.

![[Pasted image 20240306150528 1.png]]

Add Profile 버튼을 클릭하여 Display Name을 작성 후 VPN Configuration File에 위에서 수정한 Config 파일을 업로드한다. 

![[Pasted image 20240306150618 1.png]]

생성한 프로필을 선택 후 Connect 버튼을 눌러 Client VPN 엔드포인트에 접속한다.
Connected. 라고 뜨면 접속 성공이다.

![[Pasted image 20240306150643 1.png]]
<br/>

5. **온프레미스 환경 접속**

	![[Pasted image 20240306150733 1.png]]
	Site-to-Site VPN으로 연결된 온프레미스 네트워크에 IDC에 대한 허용 규칙과 라우트를 추가해주면 접속이 가능하다.

VPC 콘솔에서 VPN -> Client VPN endpoints에서 VPN 엔드포인트를 선택 후 
아래 탭에서 Authorization Rules를 추가한다.

![[Pasted image 20240306150843 1.png]]

IDC의 CIDR을 입력하여 IDC로의 접근을 허용하는 Rule을 추가해준다.

![[Pasted image 20240306150904 1.png]]

Route table 탭에서 route를 추가한다.

![[Pasted image 20240306150931 1.png]]

고가용성을 위해 두 서브넷 모두에 IDC로의 경로를 추가해준다. 

![[Pasted image 20240306152746 1.png]]

Client VPN을 구성함으로써 원격 및 재택 근무자들에게 AWS 환경 및 AWS와 연결된 온프레미스 네트워크에도 안전하게 접속할 수 있는 인프라를 제공할 수 있다.

#### **Lambda Connection Handler**
: Lambda Connection Handler를 이용하여 ==추가적인 보안 정책 및 VPN 운영 정책==을 관리하고 적용할 수 있다.

Handler는 AWS 람다 기능을 통해 구현되며, 사전에 정책을 검토하고 **정책이 통과되었을 경우에만 새 연결**을 맺게된다. 
<br/>

1. **Lambda 함수 생성**
	==지정 시간에만 접속을 허용==하는 간단한 정책을 구성하여 적용

Lambda 콘솔에서 정책에 사용 할 함수를 생성해준다.

![[Pasted image 20240306154648 1.png]]

"Author form scratch" 옵션으로 선택하고, 아래와 같이 입력해준다.
참고로 Lambda 함수의 이름은 ==**AWSClientVPN**== 접두사로 시작해야 Connection Handler로 사용할 수 있다. 

![[Pasted image 20240306154739 1.png]]

함수를 생성하고, 해당 함수의 handler에 아래의 파이썬 함수를 붙여넣는다.

```
import datetime

def is_weekday():
    week_index = datetime.datetime.today().weekday()
    if week_index < 5:
        return True
    return False

def lambda_handler(event, context):
    allow = False
    error_msg = "You are not allowed to connect on a weekend day."
    if is_weekday():
        allow = True
    return {
        "allow": allow,
        "error-msg-on-failed-posture-compliance": error_msg,
        "posture-compliance-statuses": [],
        "schema-version": "v1"
    }
```

지정된 시간(주중)에만 연결을 허용하고,
주중이 아닐 경우 "You are not allowed to connect on a weekend day" 라는 문구를 반환한다.

+) 미리 정의된 IP pool 대역대에서만 연결을 할 수 있는 코드의 예제는 다음과 같다.

```
import ipaddress

def is_public_source_ip_valid(public_ip):
    if ipaddress.ip_address(public_ip) in ipaddress.ip_network('72.21.0.0/16'):
        return True
    return False

def lambda_handler(event, context):
    public_ip = event['public-ip']
    allow = False
    error_msg = "Your IP is not allowed to establish a connection. Please contact your administrator."
    if is_public_source_ip_valid(public_ip):
        allow = True
    return {
        "allow": allow,
        "error-msg-on-failed-posture-compliance": error_msg,
        "posture-compliance-statuses": [],
        "schema-version": "v1"        
    }
```

<br/>

2. **Client VPN에 Lambda Connect Handler 적용**

VPC 콘솔로 이동하여 Client VPN Endpoint 설정을 변경한다.
정책을 적용 할 VPN Endpoint를 선택하고, Actions -> Modify Client VPN Endpoint를 선택한다.

![[Pasted image 20240306155124 1.png]]

Client connect handler 메뉴로 이동하여 "Enable client connect handler" 옵션을 On 한 뒤
ARN에서 방금 생성한 함수를 선택한다.

![[Pasted image 20240306155158 1.png]]

모든 설정을 완료하고, VPN Client 정책에 반하는 조건으로 접속 시 아래와 같은 에러 메시지가 반환된다.

![[Pasted image 20240306155221 1.png]]

