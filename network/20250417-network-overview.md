# 컴퓨터 네트워크

## 1. 컴퓨터 네트워크를 알아야 하는 이유
### 1-1. 컴퓨터 네트워크란?
- 여러 장치가 연결되어 서로 정보를 주고 받을 수 있는 통신망을 의미
- 일상적으로 사용하는 데스크톱, 노트북, 스마트폰은 대부분 주변 장치와 유무선으로 연결되어 정보를 주고 받을 수 있음
- 그렇게 연결된 장치 또한 또 다른 주변 장치와 연결되어 정보를 주고 받을 수 있음

### 1-2. 인터넷이란?
- 여러 네트워크를 연결한 '네트워크의 네트워크'
- 네트워크를 통해 서로 연결되면 주변의 장치뿐만 아니라 네트워크와 연결된 지구 반대편에 있는 장치와도 서로 정보를 주고 받을 수 있음

## 2. 네트워크 거시적으로 살펴보기
### 2-1. 네트워크의 기본 구조
- 네트워크는 노드와 노드를 연결하는 간선으로 이루어진 '그래프' 형태를 띠고 있음
- 모든 네트워크는 '노드', 노드를 연결하는 '간선', 노드 간 주고받는 '메시지'로 구성됨
- 노드는 정보를 주고받을 수 있는 장치, 간선은 정보를 주고받을 수 있는 유무선의 통신 매체를 의미

**호스트**
- 네트워크의 가장자리에 위치한 노드
- 네트워크를 통해 흐르는 정보를 최초로 생성 및 송신하고 최종적으로 수신
- 우리가 일상에서 사용하는 네트워크 기기 대부분이 여기에 속함 (예 : 서버 컴퓨터, 개인 데스크톱, 노트북, 스마트폰, 시계, 자동차, 냉장고, TV 등)

**서버**
- '어떠한 서비스'를 제공하는 호스트
- '어떠한 서비스'는 파일, 웹 페이지, 메일 등이 있음

**클라이언트**
- 서버에게 어떠한 서비스를 요청하고 서버의 응답을 제공받는 호스트
- 노트북에서 웹 브라우저를 열고 구글 웹 페이지에 접속을 시도한다면, 노트북은 클라이언트로서 구글 서버에 웹 페이지를 요청한 것임

**네트워크 장비**
- 네트워크 가장자리에 위치하지 않은 노드, 즉 호스트가 주고받을 정보가 중간에 거치는 노드
- 호스트 간 주고받는 정보가 원하는 수신지까지 안전하게 전송될 수 있도록 함
- 예 : 허브, 스위치, 라우터, 공유기 등

**통신 매체**
- 각 노드를 연결하는 간선
- 노드들을 유선으로 연결하는 유선 매체, 무선으로 연결하는 무선 매체가 있음
- 호스트와 네트워크 장비는 유무선 매체를 통해 연결됨

**메시지**
- 통신 매체로 연결된 노드가 주고받는 정보
- 예 : 웹 페이지, 파일, 메일 등

### 2-2. 범위에 따른 네트워크 분류
**LAN**
- Local Area Network의 약자로, 가까운 지역을 연결한 근거리 통신망
- 가정, 기업, 학교처럼 한정된 공간에서의 네트워크

**WAN**
- Wide Area Network의 약자로, 먼 지역을 연결하는 광역 통신망
- 멀리 떨어진 LAN을 연결할 수 있는 네트워크
- 다른 LAN에 속한 호스트와 메시지를 주고받아야 할 때 필요함
- 예 : 인터넷
※ ISP : 사용자에게 인터넷과 같은 WAN에 연결 가능한 회선을 임대하는 등의 서비스 제공하는 업체

### 2-3. 메시지 교환 방식에 따른 네트워크 분류
**회선 교환 방식**
- 호스트들이 메시지를 주고받기 전에 두 호스트를 연결한 후, 연결된 경로(='회선')로 메시지를 주고받는 방식
- 예 : 전통적인 전화망
- 장점 : 주어진 시간동안 전송되는 정보의 양이 비교적 일정함
- 단점 : 메시지를 주고받지 않으면서 회선을 점유하고 있는 경우 낭비임
 
**패킷 교환 방식**
- 메시지를 패킷이라는 작은 단위로 쪼개어 전송
- 각 패킷은 독립적으로 전송되며, 목적지에서 다시 조립됨
- 패킷은 전송하고자하는 데이터인 페이로드와 부가정보인 헤더 및 트레일러로 구성
- 장점 : 하나의 전송 경로를 점유하지 않기에 네트워크 이용 효율이 상대적으로 높음
- 단점 : 순서 보장되지 않음, 패킷 손실 가능성, 오버헤드 존재
