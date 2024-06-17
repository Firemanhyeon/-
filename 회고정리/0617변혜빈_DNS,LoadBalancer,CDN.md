---
date: 2024-06-14
---

> [!info]- 참고한 사이트
> [What is DNS?](https://aws.amazon.com/route53/what-is-dns/)
> [What is a domain name system (DNS) resolver?](https://www.lenovo.com/us/en/glossary/dns-resolver/?orgRef=https%253A%252F%252Fwww.google.com%252F)
> [재귀 DNS란?](https://www.cloudflare.com/ko-kr/learning/dns/what-is-recursive-dns/)
> [로드 밸런싱 유형](https://prohannah.tistory.com/68)
> [Open Connect Overview](https://openconnect.netflix.com/Open-Connect-Overview.pdf)

## 11. DNS 연결 관리
### 11-1. DNS (Domain Name System)
DNS는 사람이 읽을 수 있는 도메인 이름을 -> 실제 통신에 사용하는 IP 주소로 변환합니다.
DNS 서버는 `도메인`과 `IP 주소`간의 매핑을 관리하는 전화번호부와 같은 역할을 합니다.

DNS 서버는 도메인 이름을 웹 브라우저에 입력할 때 최종 사용자를 어떤 서버에 연결할 것인지를 제어하는데, 이 요청을 `쿼리` 라고 합니다.

> DNS 서버의 유형

![[DNSServerHierarchy.png]]
- **Root DNS 서버**
	- DNS 계층 구조의 최상위 레벨에 위치
	- 최상위 도메인 (TLD) 네임서버의 정보를 제공
	- 전 세계적으로 13개의 루트 네임 서버가 있다.
- **TLD (Top-Level domain) 서버**
	- 특정 최상위 도메인(.com, .net, .org) 에 대한 정보를 관리
	- 해당 TLD에 속한 도메인데 대한 권한 있는 DNS 서버의 정보를 제공
- **권한 있는 DNS 서버 (Authoritative DNS server)**
	- 특정 도메인데 대한 최종 정보를 제공하는 서버
	- 도메인 소유자가 설정한 DNS 레코드를 저장하고 관리
- **재귀 네임서버 (Recursive Nameserver)**
	- 사용자의 DNS 요청을 처리하고, 필요한 경우 다른 DNS 서버에 요청을 전달
	- 사용자가 입력한 도메인의 최종 IP 주소를 찾기 위해 필요한 모든 단계를 수행 
	- 즉, 실제 DNS 레코드를 소유하지는 않지만 열심히 찾아 줌

> DNS 서버가 트래픽을 라우팅하는 과정

![[DomainHierarchy.png]]
![[DNSRouteTraffic.png]]
1. **사용자 입력 (example.com)**
	- 사용자가 웹 브라우저에 `example.com` 을 입력합니다. 
2. **DNS 해석기(DNS Resolver)의 캐시 확인**
	- 사용자의 컴퓨터는 DNS 해석기(리졸버)에게 도메인 이름의 IP 주소를 요청합니다.
	- DNS 해석기가 자신의 캐시에 요청받은 `example.com`에 대한 IP주소가 저장되어 있는지 확인합니다. 캐시에 IP 주소가 있다면 즉시 반환합니다. 
3. **Root DNS 서버 문의**
	- _재귀 쿼리 (Recursive Query)_
		- DNS 해석기가 루트 서버(Root Server)에게 `example.com` 에 대한 쿼리를 보냅니다. 
	- _반복 쿼리 (Iterative Query)_
		- 루트 서버는 `example.com` 의 정보를 알지 못하지만, `.com` 최상위 도메인(TLD) 서버의 주소를 반환합니다. 
4. **TLD DNS 서버 문의**
	- DNS 해석기가 TLD 서버에게 example.com 에 대한 쿼리를 보냅니다.
	- TLD 서버는 요청된 도메인 `example.com`에 대한 권한있는 서버의 주소를 반환합니다. 
	- (여기서 권한 있는 서버가 바로 Route53)
5. **권한 있는 DNS 서버 문의**
	- DNS 해석기는 권한 있는 DNS 서버에게 최종적으로 `example.com` 의 IP 주소를 요청합니다.
	- 권한 있는 DNS 서버는 `example.com` 의 IP 주소를 DNS 해석기에게 반환합니다. 
6. **IP 주소 반환 및 캐싱** 
	- DNS 해석기는 최종적으로 얻은 IP 주소를 사용자에게 반환하며 , 사용자는 해당 IP 주소 를 통해 웹 서버에 접속하여 `example.com` 웹 페이지를 로드합니다.
	- 또한 이 IP 주소를 캐시에 저장하여 이후에 동일한 요청이 있을 경우 더 빠르게 응답할 수 있도록 합니다.

> DNS Resolver?
- `DNS 해석기`는 도메인 이름을 IP 주소로 변환하는 특정 유형의 DNS 서버입니다. 반면 `DNS 서버`라는 용어는 DNS 시스템에 관련된 다양한 유형의 서버를 포괄하는 광범위한 용어입니다. 
- ISP 네트워크
	- 대부분의 가정용 및 소규모 비즈니스 네트워크에서는 ISP (Internet Service Provider) 장치에서 자동으로 제공하는 기본 DNS 해석기를 사용합니다. 
	- **ISP**? KT, SKT브로드밴드, LG U+... 말그대로 인터넷 서비스를 제공하는 회사들 
- 공용 DNS 해석기
	- ISP의 DNS 해석기 대신에 공용 DNS 해석기를 사용할 수 있습니다. 
	- Google Public DNS (`8.8.8.8`, `8.8.4.4`) 혹은 Cloudflare DNS (`1.1.1.1`, `1.0.0.1`) 등
	- 이와 같은 공용 DNS 해석기는 신뢰성, 빠른 응답, 보안, 멀웨어 차단 등의 이점이 있습니다. 
- 회사 또는 조직 네트워크
	- 대규모 기업이나 조직은 자체적으로 DNS 해석기를 운영하기도 합니다. 마찬가지로 빠른 응답, 보안의 측면에서 이점이 있습니다.
- 로컬 네트워크 장비
	- - 일부 라우터 및 홈 네트워킹 장비는 내장된 DNS Resolver 기능을 가지고 있습니다.
	- 이러한 장비는 로컬 네트워크 내부에서 DNS 쿼리를 처리할 수 있으며, 필요시 외부의 상위 DNS 서버로 쿼리를 전달합니다.

> 재귀 DNS 서버?
- 재귀 DNS 조회는 한 DNS 서버에서 다른 여러 DNS 서버와 통신하여 IP 주소를 찾아내고 클라이언트에 반환하는 것입니다. 
- 즉 클라이언트는 재귀 DNS 서버에게 한 번의 요청 이후, 최종값을 반환할 때까지 위임을 하는 것입니다. 재귀 DNS 서버는 위임을 받으면 알아서 재귀적 매커니즘으로 IP주소를 찾는거죠.
- 이는 클라이언트가 조회와 관련된 각 DNS 서버와 직접 통신하는 반복 DNS 쿼리와 대조됩니다. 
- 보통의 DNS Resolver는 재귀 방
- 장점
	- 재귀 DNS 쿼리는 캐싱을 사용하기 때문에 일반적으로 반복 쿼리보다 빠르게 해결됩니다.
	- 재귀 DNS 서버는 수행하는 모든 쿼리에 대한 최종 응답을 캐시하고 TTL동안 최종 응답을 저장합니다. 
- 단점
	- 개방형 DNS 서버에서 재귀 DNS 쿼리를 허용하는 구성에서는 DNS 증폭 공겨겨 및 DNS 캐시 포이즈닝 공격에 대한 위험이 있습니다. 

> 재귀와 반복의 차이점 (쿼리 유형)
- **재귀 쿼리 (Recursive Query)** 
	1. 클라리언트가 DNS 서버 (재귀 네임 서버 혹은 DNS Resolver) 에게 특정 도메인의 IP 주소를 요청합니다. 
	2. DNS Resolver는 캐시를 확인하여 결과를 반환하거나, Root DNS 서버, TLD 서버, 권한있는 DNS 서버에 반복적으로 쿼리를 보냅니다.
	3. 최종적으로 IP 주소를 찾아 클라이언트에게 반환합니다. 
	4. 클라이언트는 중간 과정에 대해 알 필요 없이 최종 결과만 받습니다. 
- **반복 쿼리 (Iterative Query)** 
	1. 클라이언트가 DNS 서버 (로컬 DNS 서버 혹은 DNS Resolver) 에게 특정 도메인의 IP 주소를 요청합니다.
	2. 로컬 DNS 서버는 캐시에 정보가 없다면 Root DNS 서버의 주소를 클라이언트에게 반환합니다.
	3. 클라이언트는 Root DNS 서버에 쿼리를 보내고, Root DNS 서버는 TLD 서버의 주소를 반환합니다.
	4. 클라이언트는 TLD 서버에 쿼리를 보내고, TLD 서버는 권한 있는 DNS 서버의 주소를 반환합니다.
	5. 클라이언트는 권한 있는 DNS 서버에 쿼리를 보내고, 최종적으로 IP 주소를 반환합니다. 
	6. DNS Resolver는 각 단계에서 필요한 정보를 제공하는 역할을 합니다.
- 요약
	- 재귀 쿼리는 DNS Resolver에게 모든 단계 위임, 클라이언트는 한 번의 요청으로 결과를 받는다.
	- 반복 쿼리는 클라이언트가 여러 단계의 DNS 서버와 직접 통신하며, 각 단계에서 필요한 정보를 받아 다음 단계로 요청을 진행한다. 


### 11-2. Route 53
AWS에서 제공하는 확장성, 고가용성의 클라우드 DNS

> 특징 
- 고가용성: Route 53은 여러 리전에 걸쳐 분산되어 있어 DNS 인프라 장애 가능성을 최소화 합니다.
- 확장성: 트래픽이 증가함에 따라 자동으로 확장되어 많은 양의 DNS 요청을 처리할 수 있습니다. 
- 도메인 등록 기능 제공
- 트래픽 라우팅 정책 -> 가용성과 성능 최적화
	- Simple Routing : 하나의 DNS 레코드가 하나의 IP 주소를 반환
	- Weighted Routing : 여러 IP 주소에 대해 가중치를 설정하여 트래픽을 분산 
	- Geolocation Routing : 사용자와 가까운 서버에 연결하도록 트래픽을 라우팅 
	- Latency-Based Routing : 가장 빠르게 응답할 수 있는 서버로 라우팅
	- Failover Routing : 주 서버에 장애 발생시 예비 서버로 자동 라우팅 
- 헬스 체크 및 모니터링 
- AWS 다른 서비스와 통합 -> 관리 용이성, 자동화
- 보안

### 11-3. 실습: 다른 곳에서 구매한 도메인을 Amazon Route 53에 등록 하기

## 12. 로드 밸런싱
### 12-1. Load Balancing
![[LoadBalancer.png]]
- 네트워크 트래픽을 여러 서버에 분산시켜 웹 어플리케이션이나 서비스의 성능과 가용성을 향상시키는 기술입니다. 
- 이를 통해 하나의 서버에 과도한 트래픽이 집중되는 것을 방지하고, 서버 간 부하를 균등하게 분산하여 전체 시스템의 효율성을 높입니다.
- 로드 밸런싱은 특히 대규모 트래픽이 발생하는 웹 서비스에서 중요한 역할을 합니다. 
- 일종의 reverse proxy의 한 종류 !! 

> 필요성
- 고가용성
- 확장성
- 성능 향상
- 유연성

> 동작 원리
1. 클라리언트 요청
2. 트래픽 분산
3. 서버 응답

> 로드 밸런싱의 이점
- 무중단 서비스
- 효율적인 리소스 사용
- 보안 강화
- 유연한 확장성

> 로드 밸런싱의 구성 요소
- 로드 밸런서
- 백엔드 서버 : 대상 서버
- 헬스 체크
	- 서버의 상태를 모니터링하여 정상 작동 여부를 확인하는 기능입니다.
	- 로드 밸런서는 헬스 체크를 통해 장애가 발생한 서버를 감지하고 트래픽 분산 대상을 조정합니다.

> 로드 밸런싱의 종류
- 하드웨어 로드 밸런싱
	- 전용 하드웨어 장치를 사용하여 트래픽을 분산하는 방식입니다.
	- 높은 성능과 안전성을 제공하지만 비용이 많이 듭니다.
	- 보통 리눅스를 실행하는 서버일 뿐이거나 관리 툴이 포함된 임베디드 리눅스 배포
- 소프트웨어 로드 밸런싱
	- 소프트웨어 기반 솔루션을 사용하여 트래픽을 분산하는 방식입니다.
	- 비용 효율적이며 유연성이 뛰어납니다.
	- 구매한 하드웨어 서버에 설치하거나 클라우드 환경에 설
	- 예- Nginx, HAProxy
- DNS 를 사용한 로드밸런싱
	- 별도의 로드밸런서 없이 DNS 만으로 트래픽을 분산하는 기법입니다. 
	- Round Robin : 하나의 도메인 이름을 Round Robin 방식으로 N 개의 IP 주소로 변환하여 트래픽을 분산합니다. 
	- 서브 도메인 사용
	- DNS 레코드 TTL 조정
	- 지리적 DNS 로드 밸런싱

### 12-2. 로드 밸런싱 방식
#### 1. 라운드 로빈 (Round Robin)
#### 2. 가중치 라운드 로빈 (Weighted Round Robin)
#### 3. 최소 연결 (Least Connections)
#### 4. IP 해시 (IP Hash)
#### 5. URL 해시 (URL hash)
#### 6. 응답 시간 기반 (Response Time-Based)

### 12-3. Amazon Elastic Load Balancing (ELB)
AWS에서 제공하는 _관리형 로드 밸런싱_ 서비스 로, 웹 애플리케이션의 트래픽을 여러 대상(예: EC2 인턴스, 컨테이너, IP 주소)에 자동 으로 분산시켜 고가용성과 확장성을 제공합니다.

> ELB 구성 요소
- 로드 밸런서
	- 클라이언트의 요청을 수신하고 이를 여러 대상에 분산시키는 역할을 합니다.
- 리스너
	- 로드 밸런서에서 수신하는 연결 요청을 정의합니다.
	- 리스너는 포트 및 프로토콜을 지정합니다.
- 대상 그룹
	- 로그 밸런서가 트래픽을 분산시키는 대상을 정의합니다.
	- 대상 그룹은 EC2 인스턴스, 컨테이너, IP 주소 등을 포함할 수 있습니다.
- 헬스 체크
	- 대상의 상태를 모니터링하여 정상 작동 여부를 확인합니다. 
	- 헬스 체크를 통과하지 못한 대상을 트래픽 분산에서 제외합니다.

### 12-4. ELB 종류 및 유형 
#### 1. 어플리케이션 로드 밸런서 (ALB)
HTTP 및 HTTPS 트래픽을 위한 로드 밸런서로, 고급 라우팅 기능, TLS 종료, WebSocket 지원 등을 제공합니다.

> 사용 사례
- 복잡한 라우팅 요구 사항이 있는 웹 어플리케이션
- 다수의 마이크로 서비스를 사용하는 어플리케이션
- 실시간 통신이 필요한 어플리케이션

#### 2. 네트워크 로드 밸런서 (NLB)
초고속 성능과 낮은 지연 시간을 요구하는 TCP 및 UDP 트래픽을 위한 로드 밸런서입니다. 수백만 개의 요청을 처리할 수 있습니다.

> 사용 사례
- 초저지연이 필요한 금융 거래 어플리케이션
- 실시간 게임 서버
- VoIP  및 영상 스트리밍 어플리케이션 

#### 3. 게이트웨이 로드 밸런서 (GWLD)
- 서드파티 가상 어플라이언스(예: 방화벽, 침입 탐지 시스템)를 확장 가능하게 배포할 수 있도록 지원합니다.
- 네트워크 트래픽을 가상 어플라이언스로 라우팅하여 보안 및 모니터링 기능을 제공합니다. 

> 사용 사례
- 네트워크 보안 강화를 위한 서드파티 보안 솔루션 배포
- 트래픽 모니터링 및 분석을 위한 네트워크 어플라이언스 통합
- 침입 탐지 및 방지 시스템을 통한 네트워크 보호

### 12-5. 실습: ELD 구성하기 - 1대의 EC2를 복제해서 2대로 구성하고 ELD를 구성하는 실습
- ec2 자체에 준 권한 -> 이미지 추출해도, 권한은 이미지에 포함되는 것이 아님. ec2 자체의 것이기 때문에 별도임
- ec2 내부의 config에서 작성한 것 -> 해당 인스턴스 내부에 파일로 있기 때문에 이미지 추출시 당연히 함께 포함 됨

이미지? 
인스턴스의 여러 디스크에 있는 모든 구성, 메타데이터, 권한, 데이터를 저장하는 Compute Engine 리소스
일종의 템플릿, 가상화 환경에서 VM을 신속하게 배포하고 관리할 수 있음
- 운영체제 이미지
- 소프트웨어 설치 및 설정 : 특정 소프트웨어 패키지, 라이브러리, 설정 파일 등
- 가상 하드웨어 설정 : 가상 CPU, 메모리, 디스크
- 사전 구성된 설정 : 보안 설정, 네트워크 설정, 사용자 계정 등 

### 12-6. 실습: 서비스 실패 시 ELB 테스트하기
### 12-7. ALB를 Route53에 등록하기
### 12-8. 실습: HTTPS 설정하기

## 13. CDN, CloudFront
### 13-1. CDN (Contents Delivery Network)
![[CDN.png]]
- CDN은 전 세계에 분산된 서버 네트워크를 통해 사용자에게 웹 콘텐츠를 빠르고 효율적으로 전달하는 시스템입니다.
- 웹 페이지, 이미지, 동영상, 스타일시트, 자바스크립트 등 다양한 종류의 콘텐츠를 제공하는데 사용됩니다.
- 주 목적은 콘텐츠 전송 속도를 높이고, 대력폭 비용을 절감하며, 서버의 부하를 분산시키는 것입니다.
- 포인트는 분산된 엣지 서버에 캐싱을 사용한다는 것! 

> CDN 의 이점
- 속도 향상 
	- 사용자의 지리적 위치에서 가까운 서버에서 콘텐츠를 제공하기 때문에 로딩 시간이 단축됩니다.
- 서버 부하 분산
	- 트래픽이 분산되어 원본 서버의 부하가 줄어듭니다.
- 대역폭 비용 절감
	- 캐시된 컨텐츠가 반복적으로 제공되므로 원본 서버에서 발생하는 대역폭 사용량이 줄어듭니다.
- 가용성 및 안정성 향상
	- 분산된 서버 네트워크이기 때문에 장애 발생 시에도 서비스의 연속성을 보장합니다.

> CDN의 주요 구성 요소
- Edge Server
	- 사용자의 요청을 처리하고, 자주 요청되는 콘텐츠를 캐싱하여 빠르게 제공하는 서버입니다.
	- 전 세계 여러 지역에 분산되어 있습니다.
- Origin Server
	- 실제 콘텐츠가 저장되어 있는 서버입니다.
	- 에지 서버에 캐시되어 있지 않은 콘텐츠를 요청하면, 원본 서버에서 데이터를 가져옵니다.
- Caching
	- 자주 요청되는 콘텐츠를 에지 서버에 저장하여, 우너본 서버에 대한 요청수를 줄이고 응답 시간을 단축하는 기술힙니다.
  - Distribution
	  - CDN은 콘텐츠를 여러 에지 서버에 분산시켜, 사용자에게 가장 가까운 서버에서 콘텐츠를 제공하도록 합니다.

> 주요 사례
- 웹 사이트 및 웹 어플리케이션
- 비디오 스트리밍
- 소프트웨어 배포
- 전자상거래 

### 13-2. CDN의 동작 원리
Cache Hit
![[CDNCacheHit.gif]]
Cache Miss
![[CDNCacheMiss.gif]]

 > CDN의 주요 기술
 - Anycast
	 - 사용자 요청을 가장 가까운 에지 서버로 라우팅하는 네트워크 라우팅 기술입니다.
 - 헤더 기반 라우팅
	 - HTTP 헤더 정보를 분석하여 최적의 에지 서버로 요청을 라우팅합니다. 
 - 지리적 라우팅
	 - 사용자의 지리적 위치를 기반으로 요청을 가장 가까운 에지 서버로 라우팅합니다. 
 - 프로토콜 최적화
	 - TCP 및 HTTP/2와 같은 프로토콜 최적화를 통해 데이터 전송 속도를 향상시킵니다.

### [참고] CDN for Netflix
> Moving to AWS in 2009
- In 2007, Netflix started on-demand video streaming using their own data center.
- For three days in August 2008, Netflix could not ship DVDs because of corruption in their database. This incident was unacceptable. Netflix had do something,
- Netflix moved to AWS because it wanted a more reliable infrastructure.

> Why did Netflix choose AWS?
- Didn't want to do any undifferentiated heavy lifting.
- _Undifferentiated heavy lifting_ is those things that have to de done but don't provide any advantage to the core business of providing a quality video-watching experience.
- It took more than eight years for Netflix to complete the process of moving from their data centers to AWS.
- During that period, Netflix grew its number of streaming customers eightfold.

> CDN for Netflix
- Netflix built its simple CDN in five different locations within the United States in 2007
	- 36 Million Members in 50 different countries
- Started 3rd party CDN in 2009
	- Focusing on user satisfaction and experience
- Open Connect - Netflix's CDN (2011-2)
	- A dedicated CDN solution to maximize network efficiency
	- Cut the cost and service quality enhancing (transcoding, client control- AWS, CDN- Netflix)
	- It is possible because Netflix knows about user - preferences and location
	- Locating OCA all around the world near the user
		- Large OCA - store Netflix's entire video catalog
		- Smaller OCA - filled with video every day, during off-peak hours, using a process Netflix calls proactive caching

> OCA (Open Connect Appliances) and Content Caching

넷플릭스가 개발한 맞춤형 CDN 아키텍처
![[OCA.png]]
- Locating OCA all around the world
	- More than 1000 locations
	- Does not operate Netflix's backbone network
	- Locate OCA in ISPs or IXPs
- Netflix is the data-driven company
	- Popularity of the content in an certain location
	- The OCA works as the cache of the contents
	- Proactive Caching OCA to large OCA, Large OCA to Smaller OCA
	- Every night OCA prepares the content for the users before they click the content

### 13-3. CDN 캐싱 방식의 종류
#### 1. 시간 기반 캐싱 (Time-based Caching)
- 콘텐츠가 일정 기간 동안 캐시에 저장되도록 설정하는 방식
- 설정된 시간이 만료되면 원본 서버에서 새로운 콘텐츠를 가져와 갱신
- 주기적으로 업데이트되는 콘텐츠에 적합
- TTL (Time To Live)

#### 2. 조건부 요청 캐싱 (Conditional Request Caching)
- 클라이언트가 서버에 요청 시, 해당 콘텐츠가 수정되었는지 여부를 확인하는 방식
- 콘텐츠가 변경되지 않았을 경우, 304 Not Modified 코드와 함께 콘텐츠를 그대로 사용하도록 함
- ETag (Entity Tag) 
	- 컨텐츠의 버전을 식별하는 태그, 서버와 클라이언트의 ETage를 비교하여 변경 여부 판단
- Last-Modified 
	- 콘텐츠의 마지막 수정 시간
- 불필요한 데이터 전송을 줄이지만 추가적인 헤더 정보를 처리해야 하므로 구현이 복잡할 수 있음

#### 3. 캐시 우회 (Cache Bypass)
- 특정 조건을 만족하는 요청에 대해 캐시를 사용하지 않고 원본 서버에서 직접 콘텐츠를 가져오는 방식
- 실시간이나 자주 변경되는 콘텐츠에 유용
- Query String 
	- URL에 포함된 쿼리 스트링을 기반으로 캐시 우회가 설정될 수 있음
- 실시간 데이터와 같이 자주 변경되는 콘텐츠를 항상 최신 상태로 유지
- 원본 서버에 대한 요청의 증가로 인한 부담

#### 4. 분할 캐싱 (Partitioned Caching)
- 동일한 URL 이지만 다양한 버전의 콘텐츠를 캐시하는 방식
- 다국어 웹사이트나 사용자 맞춤형 콘텐츠에 유용
- 헤더 기반 캐싱
	- HTTP 헤더 정보를 기반으로 콘텐츠를 분할하여 캐싱 
	- 예) `Accept-Language` 헤더를 사용하여 언어별로 콘텐츠를 캐시
- 사용자 맞춤형 콘텐츠 제공 가능
- 캐시 저장소릐 사용량 증가가

#### 5. 계층적 캐싱 (Hierarchical Caching)
- 여러 계층의 캐시 서버를 두어 콘텐츠를 단계적으로 캐싱하는 방식
- 첫 번째 계층의 캐시 서버가 요청을 처리하지 못할 경우, 다음 계층의 캐시 서버가 요청을 처리
- Edge Cache
	- 사용자의 가까운 위치에서 콘텐츠를 제공하는 1차 캐시 서버
- Origin Cache
	- 원본 서버에 가까운 위치에서 콘텐츠를 제공하는 2차 캐시 서버
- 캐시 서버 간의 부하를 분산 
- 콘텐츠 전달 속도를 극대화
- 계층이 많아질수록 관리의 복잡성도 증가

#### 6. 네거티브 캐싱 (Negative Caching)
- 요청한 컨텐츠가 존재하지 않거나 오류가 발생했을 때, 그 결과를 일정 시간 동안 캐시에 저장하는 방식
- 잘못된 요청이 반복적으로 원본 서버로 전달되는 것을 방지함
- 원본 서버의 불필요한 부하를 줄이고 응답 시간 개선
- 실제로 콘텐츠가 존재하지만 일시적인 오류로 인해 캐시에 저장된 경우, 사용자가 콘텐츠에 접근할 수 없는 상황이 발생할 수 있음

### 13-4. Amazon CloudFront
AWS의 CDN 서비스

### 13-5. CloudFront의 특징
> 특징
- 글로벌 분산 네트워크
	- 당연함 전 세계에 AWS 서버 존나 많음
- 원본 서버 통합
	- 원본 서버로서 S3, EC2, ELB, Lambda와 같은 AWS 서비스 뿐만 아니라 사용자 지정 원본 서버를 사용할 수 있음
- 동적 및 정적 콘텐츠 제공
	- 정적 콘텐츠 (이미지, CSS, JS) 뿐만 아니라 동적 콘텐츠 (API 응답, HTML 파일)도 효율적으로 제공
- 보안 기능
	- SSL/ TLS 암호화
	- AWS WAF 통합
	- DDoS 보호 : AWS Shied
	- 제한된 콘텐츠 접근 : 서명된 URL 및 쿠키를 사용해 특정 사용자만 접근 가능
- 지리적 제약
	- 특정 지역의 사용자에게만 콘텐츠를 제공하거나 차단 가능
- 실시간 분석 및 모니터링
	- Amazon CloudWatch와 통합하여 실시간 모니터링 가능

### 13-6. CloudFront의 주요 기능
### 13-7. 실습: Amazon S3 정적 웹 사이트 구성하기
### 13-8. 실습: CloudFront 웹 배포 생성 후 S3와 연결하기

cloud front -> CDN의 역할을 한다...
우리는 원본 서버를 s3로 설정했음.
cloud front가 해당 내용을 캐싱하고 있다면 실제 서버에 접속하지 않고 캐싱한 정보를 사용..! 



