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
* addOn
	* Prometheus : 매트릭을 쌓고 있는 Stateful Service
	* Grafana : Prometheus의 매트릭을 시각화
	* Jaeger : 분산 추적 시스템
	* Kiali : Jaeger, Istio의 구성요소를 시각화
		* Workload : 서비스 간의 이동
* CLI
	* 이슈 발생시
		* 트래픽이 많다면 파드별 accesslog는 꺼져있을 수 있음
		* 해당 파드에 접근해 istio-proxy의 로그 on
		* istiod와 istio-proxy 로그 분석 및 프록시 상태/config 확인
	* 명령어
		* kubectl exec -it {POD} -c {container} -- {command}
		* curl -XPOST localhost:15000/logging?http=debug// 디버깅모드 동적설정, 해당 istio-proxy에 들어가야함
		* istioctl proxy-status : SYNC 전체 상태를 확인할 수 있는 명령어
			* Discovery Service
				* CDS : Cluster Discovery Service
				* EDS : Endpoint Discovery Service
				* LDS : Listener Discovery Service
				* RDS : Route Discovery Service
				* ADS (Aggregation Discovery Service)
					* istiod와 지속적으로 통신
					* 위의 4가지를 통합하여 싱글 스트림으로 해결
			* 상태
				* SYNCED : 잘됨
				* NOT SENT : 업데이트 안됨
				* STALE : envoy가 전달했는데, 응답이 없음
		* istioctl proxy-config listener productpage-v1-8b588bf6d-4nztw.default // 해당 파드의 istio-proxy의 리스너 정보 확인,
		  cluster, route 등도 확인 가능

# Envoy

* 개념
	* Upstream
		* Envoy가 요청을 포워딩해서 열결하는 백엔드 네트워크 노드
		* 사이드카일땐 application app, 아닐땐 원격 백엔드
	* Downstream
		* Envoy 프록시에 연결하는 클라이언트
	* Clusters
		* Envoy가 연결하는 Upstream의 집합
		* Endpoint의 세트
		* Envoy가 트래픽을 포워드할 수 있는 논리적인 서비스
	* Endpoints
		* Upstream의 IP 주소와 포트
		* 네트워크 노드로 클러스터로 그룹핑
	* Listener
		* IP/port에 바인딩하고, 요청처리 측면에서 다운스트림을 조정하는 역할
	* Routes
		* 트래픽을 클러스터로 라우팅하는 방법을 정의
	* Filter
		* 리스너로부터 서비스에 트래픽을 전달하기까지 요청 처리 파이프라인
* x Discovery Service
	* 동적으로 각 영역의 설정을 로드하는 방법
	* Management 서비스와 통신하면서 configuration과 통신
	* gRPC,REST 모두 지원

# Istio Proxy

### [VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/)

* spec.hosts에 처리할 트래픽 hosts 등록
* L7 패스별 라우팅
* 목적지에 대한 정책이 들어가는 곳이 때문에 매우 중요한 리소스
* envoy의 RouteConfig 역할을 메인으로 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfos-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway # use istio default controller , 게이트웨이 설정
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts: # hosts에 처리할 트래픽 hosts 등록
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfos
spec:
  hosts:
  - "*"
  gateways:
  - bookinfos-gateway # gateway 설정
  - mesh # 사이드카에 트래픽이 모두 mesh로 이동함, internal traffic으로 변경
  http: # http L7 패스별 라우팅
  - match: 
#	- header: 헤드 매칭도 가능 
#		end-user:
#		  - exact: jason
    - uri:
        exact: /productpage #exact : 정확히 일치하는 패스
    - uri:
        prefix: /static #prefix : 접두사로 시작하는 패스
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
#	- uri:
#		regex: /product/[0-9]+ 정규표현식
#    rewrite:
#      uri: /v1/bookRatings uri 변경 
#    headers: # 헤더 추가
#      request:
#        add:
#          test: "1234567890" 
    route:
    - destination:
        host: productpage
        port:
          number: 9080
#	- route:
#		- destination: 위임도 가능 
#			host : productpage.v1.svc.cluster.local

```

### [DestinationRule](https://istio.io/latest/docs/reference/config/networking/destination-rule/)

* 클라이언트 사이드 Load Balancing
* 서비스 엔드포인트를 비정상으로 표시하는 조건 가능
* L4, L7 커넥션 풀 설정
* 서버의 TLS 설정
* envoy로 치면 cluster, endpoint 쪽
* TrafficPolicy
  * loadBalancer
	* L4 로드밸런싱
	* roundRobin, random, weightedLeastRequest, maglev
  * connectionPool
    * L4 커넥션 풀 설정
  * outlierDetection
    * 서비스 엔드포인트를 비정상으로 표시하는 조건 설정
  * tls
    * 서버의 TLS 설정
  * portLevelSettings
    * 포트별 설정
* 서킷 브레이커 역할도 들어있기 때문에, 케파에 대한 확인없이 적게 적용하면 장애로 이어질 수 있음.
  * Newtork flag : UO(Upstream overflow)
  * grep 할때 overflow는 보통 서킷브레이커와 관련된 플래그 
* 우선 마이그레이션 작업이라면 Virtual Service (VR) -> Destination Rule (DR) 적용 순으로천천히 적용하는 것도 방법
* 왠만하면 러프하게 잡아두고 각 서비스 담당자가 적절히 조절하도록 가이드하는 것도 좋아보임.

```
kubectl exec {{pod_name}} -c istio-proxy -- pilot-agent request GET stats | grep overflow // overflow 확인
kubectl port-forward pod/{{pod_name}} 15000:15000 -n default // istio-proxy의 15000 포트로 포워딩
```

```yaml

```

### (Gateway)[https://istio.io/latest/docs/reference/config/networking/gateway/]

* HTTP/TCP 연결을 수신하는 메쉬 로드밸런서
* 노출되어야하는 포트 집합
* 사용할 프로토콜 유형
* 로드밸런서 SNI구성 tls등을 설명
* envoy로 치면 listener 쪽
* 노출될 호스트와 포트매핑이 메인
* L7구성보다 L4구성에 초점이 있음
* 노출할 도메인들에 대해서 gateway끼리 충돌하지 않도록 잘 설정
* 와이들 카드도메인도 좋지만, namespace 분리해서 명확히하는것이 좋음 
* VirtualService와 바인딩 상태 꼭 확인이 필요
* VirtualService에 여러개의 GW를 바인딩 할 순있으나, 트래픽이 어려움으로 mesh+1 정도로만 사용하는 것이 좋음
* 숨겨진 mesh gateway라는 암묵적인 gateway를 가짐
* 메시 게이트웨이는 모든 사이드카
* 실제로는 kube-proxy 조차 타지 않음, ingress에 따라 sidecar로 바로 감 

### (Service Entry)[https://istio.io/latest/docs/reference/config/networking/service-entry/]

* 메쉬 내에 네트워크를 수동으로 등록
* istio 내에서 어플리케이션이 접촉하는 모든앱에 대해 서비스 레지스트리의 일부로 노출
* 외부리소스에 대한 설정
* 내부 Static 라우팅에 대한 설정도 포함 
* 기본적으로 서비스엔트리는 모든 서비스/네임스페이스에 내보내짐
* exportTo옵션으로 경계 설정도 중요함
* 사용처 
  * AWS Cloud Stateful 리소스, 
  * Virtual Machine 
  * Legacy, PoC 시스템

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: googleapis
spec:
  hosts: # DNS 타입, endpoints 리스트가 없다면 DNS 서비스처럼 쓰임 , 만약 쿠버네티스랑 hostname이 같다면, 기존 k8s리소스에 데코레이터처럼 동작
  - www.googleapis.com
  exportTo: # exportTo옵션으로 경계 설정도 중요함
  	- "." # 현재 폴더위치에서만 export 
  ports:
  - number: 80
    name: http
    protocol: HTTP
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```

명령어

```bash
// 확인하기 
istioctl pc plusters {podname}.{namespace} --port {port} --direction outbound -o json // 해당 pod의 outbound cluster 정보 확인
```


# Istio 