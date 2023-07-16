# DevOps

* 모놀리틱
	* 장점
		* 기술의 단일화
		* 관리/운영에 유리
	* 단점
		* 뎌러 언어/기술스택을 가져가기 어려움
		* 빌드/배포 시간이 오래걸림
		* 모든 전체 로직을 알고있는 사람이 필요함
		* 스케일링에 문제가 많음
		* 의존성이 커서 잘못된 코드 배포시 서버 전체 장애 확률이 높음
* 마이크로 서비스 아키텍쳐
	* 장점
		* 기술스택의 다양화/해당팀의 전문성과 개발속도가 빨라짐
		* 적절한 기술의 적절한 배치
		* 전체 장애가 아닌 부분 장애
	* 단점
		* 운영 관점에서는 여러 기술을 다뤄야함
		* 요청 추적이 어려워짐
		* 각 서비스별로 방어로직이 적을 때 장애 여파
		* 보안, 통제가 힘들어짐
	* 대책
		* 서킷브레이커 패턴의 Hystrix
		* 서비스 디스커버리 패턴의 Eureka
		* 모니터링 서비스 Turbine
		* API-Gateway패턴의 Zuul
		* 서비스 메시의 등장
			* 컨테이너 오케스트레이션 툴 kubernetes의 등장
			* 여러 사이드카 컨테이너를 함께 띄울 수 있는 Pod
			* API기반 무중단 리로드가 가능한 proxy 기술의 발달
				* Site Releative Engineering - 일관된 서비스 네트워킹 정책 관리
* SRE (Site Reliability engineering)
	* 사이트 안정성에 무게중심
	* 가용성에 대한 목표 정의
	* 장애 발생에 대한 대응/분석/부검
	* Metrics & Monoitoring
		* SLI (Service Level Indicator) 지표를 정의
		* SLO (Service Level Objective) : SLI 지표들의 Goal
	* Capacity Planning
		* 시스템자원의 확보
		* 자원 활용의 효율성 / 소프트웨어 성능 / 시스템 안정성
	* Change Management
		* 변경사항의 추적
		* 변경사항의 테스트
		* 변경사항의 롤백
	* Emergency Response
		* 장애 발생시 대응
		* 장애 발생시 복구
	* Culture
		* 노가다 Operation관리

# Istio & Envoy

* 프록시 사이드카 패턴: 각 서비스 별로 프록시를 주고, 프록시끼리 통신
* Kubernetes 에서 pod는 하나 이상의 컨테이너 그룹이며, 이 그룹은 스토리지 및 네트워크를 공유함.
* 컨트롤 플레인과 데이터 플레인
	* 컨트롤 플레인을 이용해 마이크로서비스들의 프록시를 정책 관리
	* 각 마이크로 서비스들은 소유한 프록시를 활용
		* 세밀한 네트워크 정책 수정
* 컨트롤 플레인 -> istiod
* 데이터 플레인 -> istio-proxy (envoy)
* 프록시 : 중개 서버, 뒤쪽에 시스템 구조를 숨기고, 변경할 수 있음. 로드밸런싱 등의 기능을 수행할 수 있음.
* Envoy Proxy
	* C++로 작성된 L7 프록시
	* 네트워크의 투명성을 목표
	* 다양한 필터체인 지원
	* L3/L4 필터, HTTP L7 필터
	* 동적 configuration API 제공
	* 기능이 많음
	* CNCF/ 벤더가 없음.
* Istio-rpoxy
	* Envoy Proxy를 기반으로 한 Istio의 프록시(Golang 기반)
	* envoy를 활용해 서비스앱의 사이드카 컨테이너로 활용
	* 중앙 컨트롤 플레인인 istiod와 통신하고, 서비스 앱의 트래픽을 정해진 정책/필터에 따라 처리
	* 관찰가시성을 위한 메트릭 제공
	* API 기반 Hot Realod 기능 
* Istiod는 새로운 배포나 엔드포인트가 갱신이 되면, Istio Proxy에게 갱신된 정보를 전달함.

# kubenetes 리소스

* Pod 
  * 컨테이너의 묶음. 스토리지 및 네트워크를 공유함(한개의 ip)
  * 하나 이상의 컨테이너 그룹
  * 보통 Pod 자체를 define해서 apply하지 않고, Deployment/replicaset라던지, 다른 쿠버네티스 리소스가 배포의 최소 단위로 사용
* ConfigMap
  * 키-값으로 기밀이 아닌 데이터를 저장하는데 사용하는 API
  * 파드에 환경변수 매핑 or 데이터 마운트용
* Secret
  * 기밀
* Deployment
  * 파드의 의도하는 상태를 설명
  * 래플리카셋의 설명
  * .spec.replicas. 원하는 파드 갯수
  * .spec.selector. 디플로이먼트가 관리할 파드를 찾는 방법
  * template은 파드들에 공통적으로 들어갈 템플릿 
* Service
  * 파드 집합에서 실행중인 애플리케이션을 네트워크 서비스로 노출하는 추상화 방법
  * .spec.selector 연결된 라벨 매핑
  * port : 서비스가 노출하는 포트
  * targetPort : 파드가 노출하는 포트

# Istioctl

* istioctl profile diff : 기본과 차이
* operator : istio를 관리하는 컨트롤러
* Pod / Sidecar container  
  * 파드내 네트워크 인터페이스를 컨테이너끼리 공유
  * iptables를 조작하여 inbound traffic과 outbound traffic을 istio-proxy가 선처리
  * 서비스 앱의 세밀한 네트워킹 설정에 대한 책임을 가져옮 
  * 커맨드 : istioctl kube-inject -f deployment-nginx.yaml | kubectl apply -f -
  * 네임스페이스 : kubectl label namespace default istio-injection=enabled -- // 해당 네임스페이스에 뜨는 모든 파드는 사이드카가 자동으로 injection
    * 제거 명령어의 경우, kubectl label namespace default istio-injection- -- // 해당 네임스페이스에 뜨는 모든 파드는 사이드카가 자동으로 제거
    * sidecar.istio.io/inject: "false" // 해당 파드에만 사이드카가 자동으로 제거
* Host 
  * 네트워크가 할당된 네트워크 노드
* Proxy 
  * 보안강화, 확장성, 유연성, 로드밸런싱
  * Nginx, Envoy..
* Gateway
  * 프로토콜 및 Gateway prot 설정
  * 받고자하는 hosts 등록가능
  * TLS 관련 옵션 추가 가능
  * 실행하고자하는 워크로드 proxy에 설정도 가능 
* VirtualService
  * spec.hosts에 처리할 트래픽 hosts 등록
  * L7 패스별 라우팅
  * 목적지에 대한 정책이 들어가는 곳이기 떄문에 매우 중요한 리소스 