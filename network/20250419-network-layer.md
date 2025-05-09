# 네트워크 계층
## 1. LAN을 넘어서는 네트워크 계층
### 1-1. 데이터 링크 계층의 한계
- **첫째, 물리 계층와 데이터 링크 계층만으로는 다른 네트워크까지의 도달 경로를 파악하기 어렵다.**
  - 물리 계층과 데이터 링크 계층은 같은 네트워크(LAN) 안에서만 통신 가능하다. 즉, 다른 네트워크로 가는 길은 알 수 없다.
  - LAN A에 있는 컴퓨터가 LAN B에 있는 컴퓨터에게 데이터를 보낼 때, 데이터는 여러 라우터와 네트워크를 거쳐야 한다.
  - 이때 어떤 길로 보내야 가장 빠르고 정확할지 최적의 경로를 결정하는 것을 **라우팅**이라고 한다.

- **둘째, MAC 주소만으로는 모든 네트워크에 속한 호스트의 위치를 특정하기 어렵다.**
  - MAC주소는 호스트를 식별하지만, 위치를 알 수 없기 때문에 전 세계 네트워크 간 데이터 전송에는 부적합하다.
  - 이때 네트워크 계층의 IP 주소를 사용해 논리적 주소로 위치를 지정한다.
  - 이 구조 덕분에 라우터가 목적지 네트워크를 인식하고 경로 선택(라우팅)이 가능하다.
 
### 1-2. 인터넷 프로토콜
- **IP주소 형태**
  - IPv4는 4바이트(=32비트)이고 각 바이트(8비트)를 10진수로 표현한다.(0~255 범위)
  - 각 10진수는 점(.)으로 구분되며, 점으로 구분된 8비트를 옥텟(octet)이라고 한다.
- **IP의 기능**
  - **IP 주소 지정** : IP 주소를 바탕으로 송수신 대상을 지정하는 것을 의미한다.
  - **IP 단편화** : 전송하고자 하는 패킷의 크기가 MTU라는 최대 전송 단위보다 클 경우, 이를 MTU 크기 이하의 복수의 패킷으로 나눈다.
 
#### 1-2-1. IPv4
- IPv4 패킷 헤더 중 식별자, 플래그, 단편화 오프셋 필드는 IP 단편화 기능에 관여하고, 송신지 IP 주소, 수신지 IP 주소는 IP 주소 지정 기능에 관여한다.
- **식별자** : 하나의 데이터가 여러 조각으로 단편화될 경우, 이 조각들은 원래 한 패킷이었다는 것 식별하는 번호이다.
- **플래그** : 총 3비트로 구성된다. (Reserved - 항상 0 , DF - Don't Fragment, MF - More Fragments)
- **단편화 오프셋** : 조각들이 원래 데이터의 어느 위치에 속했는지 알려준다.
- **TTL(Time To Live)** : 패킷이 영원히 네트워크를 떠돌지 않도록 수명을 제한한다. 라우터를 한 번 지날 때마다 1씩 감소하며 TTL=0일 때 해당 패킷은 폐기된다.
- **프로토콜** : IP 패킷 안에 어떤 상위 계층 프로토콜의 데이터가 들어있는지 나타낸다.
- **송신지 IP 주소** : 이 패킷을 보낸 장치의 IP 주소이다. 응답을 보낼 때 이 주소로 돌려보낸다.
- **수신지 IP 주소** : 이 패킷이 도착해야 할 목적지 장치의 IP 주소이다. 라우터는 이 주소를 보고 경로를 결정한다.

#### 1-2-2. IPv6
- 이론적으로 할당 가능한 IPv4 주소는 총 2^32 개로 약 43억 개이다.
- 결국 IPv4의 주소의 총량은 고갈될 수 있으므로 이런 이유에서 등장한 것이 IPv6이다.
- IPv6 주소는 16바이트(=128비트)이며, 각 2바이트(16비트)를 16진수로 표현해서 :으로 구분한다.
- **다음 헤더(next header)** : 해당 패킷 다음에 오는 상위 계층 프로토콜이나 확장 헤더 종류를 지정하는 필드이다.
- **홉 제한** : 패킷이 거칠 수 있는 최대 라우터 수를 의미한다. **IPv4의 TTL(Time To Live)**과 똑같은 역할이다.

### 1-3. ARP(Address Resolution Protocol)
- IP 주소를 MAC 주소로 바꿔주는 프로토콜이다.
- 실제 네트워크에서 데이터를 보내려면 MAC 주소가 있어야 패킷이 정확히 그 장치에 도착할 수 있다.
- ARP는 “같은 네트워크 안에서 IP를 가진 장치의 MAC 주소”를 알아내기 위한 도구이며, 다른 네트워크로 보낼 때는 라우터의 MAC 주소를 알아내는 데 사용된다.

- **과정**
- 예: 192.168.0.10이 192.168.0.20에게 데이터 보내고 싶음
- 1. MAC 주소를 모름 : 192.168.0.20의 MAC 주소를 모르니까 ARP Request를 브로드캐스트로 보낸다.
  2. 응답 수신 : 목적지 IP를 갖고 있는 장비가 ARP Reply로 응답한다.
  3. 결과 저장 : MAC 주소를 기억해두고, 실제 데이터는 이 MAC 주소를 목적지로 전송한다. ARP 캐시 테이블에 일정 시간동안 저장된다.
 
## 2. IP 주소
### 2-1. 네트워크 주소와 호스트 주소
- IP 주소에서 네트워크 주소와 호스트 주소를 구분하는 범위는 유동적일 수 있다.
- 그렇다면 네트워크 주소와 호스트 주소의 크기는 각각 어느 정도가 적당할까?
- 호스트 주소 공간을 크게 할당하면 호스트가 할당되지 않은 다수의 IP 주소가 낭비될 수 있다.
- 호스트 주소 공간을 작게 할당하면 호스트가 사용할 IP 주소가 부족해질 수 있다.
- 이런 고민을 해결하기 위해 IP 주소의 클래스라는 개념이 등장한다.

### 2-2. 클래스풀 주소 체계
- 클래스를 기반으로 IP 주소를 관리하는 주소 체계를 의미한다.
- 클래스는 네트워크 크기에 따라 IP 주소를 분류하는 기준이다.
- 클래스를 이용하면 필요한 호스트 IP 개수에 따라 네트워크 크기를 가변적으로 조정해 네트워크 주소와 호스트 주소를 구획할 수 있다.
- 클래스별 네트워크의 크기가 고정되어 있기에 여전히 다수의 IP 주소가 낭비될 가능성이 크다.

### 2-3. 클래스리스 주소 체계
- 클래스 개념 없이 네트워크의 영역을 나누어서 호스트에게 IP 주소 공간을 할당하는 방식이다.
- **서브넷 마스크** : 클래스리스 주소 체계에서 네트워크와 호스트를 구분짓는 수단이다. IP 주소 상에서 네트워크 주소는 1, 호스트 주소는 0으로 표기한 비트열을 의미한다.
- 서브넷 마스크를 이용해 클래스를 원하는 크기로 더 잘게 쪼개어 사용하는 것을 서브네팅이라고 한다.
- 주로 CIDR 표기법을 사용한다. 이는 'IP 주소/서브넷 마스크상의 1의 개수'로 표기하는 형식이다.

### 2-4. 공인 IP 주소와 사설 IP 주소
**공인 IP 주소** 
- 전 세계에서 고유하게 사용되는 IP 주소. 즉, 인터넷에 직접 노출되어 통신 가능한 주소
**사설 IP 주소**
- 내부 네트워크에서만 사용하는 IP주소. 인터넷에서는 사용 불가, 외부와 통신하려면 NAT 필요
- 사설 IP 주소의 할당 주체는 일반적으로 라우터이다.
- 할당받은 사설 IP 주소는 해당 호스트가 속한 사설 네트워크 상에서만 유효한 주소이다.
- 사설 IP 주소를 사용하는 호스트가 외부 네트워크와 통신하려면 NAT를 통해 IP 주소를 변환한다. 

### 2-5. 정적 IP 주소와 동적 IP 주소
**정적 할당**
- 호스트에 직접 수작업으로 IP 주소를 부여하는 방식. 이렇게 할당된 IP 주소를 정적 IP주소라고 부른다.

**동적 할당**
- 호스트에 IP 주소가 동적으로 할당되는 방식.
- 동적 IP 주소는 사용되지 않을 경우 회수되고, 할당받을 때마다 다른 주소를 받을 수 있다.

**DHCP(Dynamic Host Configuration Protocol)**
- 네트워크에 접속한 장비(클라이언트)에게 자동으로 IP 주소와 네트워크 설정 정보를 할당해주는 프로토콜
- IP 주소를 할당받고자 하는 호스트(이하 클라이언트)와 해당 호스트에게 IP 주소를 제공하는 DHCP 서버 간에 메시지를 주고받음으로써 이루어진다.
- IP 주소를 할당받는 과정은 총 4단계로 이루어져 있다.
  - 1. DHCP Discover(클라이언트 -> DHCP 서버)
       - 클라이언트는 네트워크에 처음 접속할 때 자신의 IP가 없는 상태이다.
       - 이때, **"나에게 IP 줄 DHCP 서버 누구야?"**라는 메시지를 **브로드캐스트(255.255.255.255)**로 보낸다. 이 메시지를 DHCP Discover라고 한다.
       - 출발지 IP는 아직 없으므로 0.0.0.0, 목적지 IP는 브로드캐스트 주소이다.
    2. DHCP Offer(DHCP 서버 -> 클라이언트)
        - 이 메시지를 받은 DHCP 서버는 "내가 줄 IP는 이거야" 하고 제안한다. 이 제안 메시지가 DHCP Offer이다.
        - 포함 내용 : 할당 가능한 IP 주소, 서브넷 마스크, 게이트웨이 주소, DNS 서버 주소, 임대 기간 등
    3. DHCP Request(클라이언트 -> DHCP 서버)
        - 클라이언트는 여러 DHCP 서버 중 하나를 선택해 **"방금 받은 이 IP 쓰겠습니다"**라는 응답을 보낸다. 이 메시지를 DHCP Request라고 하며, 일반적으로 브로드캐스트로 전송된다.
        - 선택되지 않은 다른 DHCP 서버는 응답을 중단합니다.
    4. DHCP Acknowledgment(이하 DHCP ACK) (DHCP 서버 -> 클라이언트)
        - DHCP 서버는 클라이언트가 선택한 IP 주소를 최종 승인하고, DHCP ACK 메시지를 보낸다.
        - 이로써 클라이언트는 해당 IP를 임대받고 네트워크에 정상적으로 접속하게 된다.

    **왜 DHCP Request를 브로드캐스트로 보내는가?**
    - IP가 없는 상태에서 유니캐스트 통신은 불가능하기 때문이다.
    - 네트워크에 존재하는 모든 DHCP 서버가 이 메시지를 듣고 자신이 선택되었는지를 판단해야 하므로 브로드캐스트가 필요하다.

## 3. 라우팅
- 네트워크에서 패킷이 출발지에서 목적지까지 도달하는 경로를 결정하는 과정

### 3-1. 라우터
- 서로 다른 네트워크 간을 연결하고, 목적지에 따라 데이터 패킷의 경로를 결정하는 장치이다.
- 패킷을 받아서 목적지 주소를 확인하고 라우팅 테이블을 참고하여 다음 홉(next hop)으로 전달한다.

### 3-2. 라우팅 테이블
- 라우터나 호스트가 어떤 IP 패킷을 어디로 보낼지를 결정하는 기준표이다.
- 구성 요소
  - Destination : 목적 주소(예: 192.168.10.0/24)
  - Netmask : 네트워크 서브넷 마스크
  - Gateway : 다음 홉 라우터의 IP
  - Interface : 패킷을 보낼 네트워크 인터페이스
  - Metric : 우선순위(작을수록 우선)
 
### 3-3. 정적 라우팅(Static Routing)
- 관리자가 직접 라우팅 정보를 설정
- 변화가 적고 구조가 단순한 네트워크에 적합
- 장점: 예측 가능하고 안정적
- 단점: 변경 시 수동 수정 필요, 대규모 환경에 부적합

### 3-4. 동적 라우팅(Dynamic Routing)
- 라우터가 라우팅 프로토콜을 사용해서 스스로 경로를 학습하고 갱신
- 네트워크가 변해도 자동으로 라우팅 테이블 업데이트
- 장점: 유지보수 용이, 대규모 네트워크에 적합
- 단점: 설정 복잡, 약간의 오버헤드

### 3-5. 라우팅 프로토콜(Routing Protocol)
- 라우터끼리 정보를 주고받고 최적의 경로를 결정하기 위해 사용하는 통신 규약이다.
- 주요 프로토콜 분류
  - 거리 벡터(Distance Vector) : 홉 수 기반, 단순하지만 느림
  - 링크 상태(Link State) : 토폴로지 전체 파악, 빠르고 정밀
  - 패스 벡터(Path Vector) : 인터넷 경로 설정, 대규모 ISP용
  - 하이브리드 : Cisco 독자 프로토콜, 거리 + 링크 상태 혼합

**홉** : 라우팅 도중 패킷이 호스트와 라우터 간에, 혹은 라우터와 라우터 간에 이동하는 하나의 과정. 즉 패킷은 '여러 홉을 거쳐' 라우팅될 수 있다.

**토폴로지(Topology)** : 네트워크에서 장치들이 어떻게 연결되어 있는지, 그 구조와 형태를 시각적으로 표현한 개념. 
즉, 컴퓨터, 스위치, 라우터 같은 장비들이 물리적으로 또는 논리적으로 어떻게 배치되고 연결되어 있는지를 나타내는 설계도
