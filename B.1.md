# 서킷 브레이커 패턴 (Circuit Breaker Pattern) 실전 사례
## 서킷 브레이커 패턴 개요 (요약)

* **목적**: 실패가 반복되는 서비스를 일정 시간 차단하여, 전체 시스템에 영향을 주는 것을 방지
* **상태**: Closed → Open → Half-Open의 세 가지 상태를 순환

  * Closed: 정상 통신
  * Open: 일정 시간 동안 요청 차단
  * Half-Open: 일부 요청만 테스트로 허용

---

## 실전 사례 1: **Netflix – Hystrix 기반 장애 억제**

### 시나리오

Netflix는 수천 개의 마이크로서비스로 구성된 대규모 시스템을 운영하며, 서로 다양한 API 호출로 연결되어 있습니다.

### 문제

어떤 특정 마이크로서비스(예: 추천 서비스)가 느려지거나 실패하면, 이를 호출하는 다른 서비스들도 연쇄적으로 지연 또는 장애 발생.

### 해결

Netflix는 **Hystrix**라는 라이브러리를 사용해 서킷 브레이커를 구현했습니다.

* 실패율이 일정 비율(예: 50%) 이상일 경우 해당 서비스로의 호출을 일시적으로 차단
* 차단 기간 후 일부 요청을 보내 서비스 회복 여부 테스트 (Half-Open)
* 회복되면 정상화(Closed), 아니면 다시 차단(Open)

### 결과

* 장애 전파 방지
* 사용자에게 degraded fallback(예: “추천 서비스는 잠시 이용할 수 없습니다”) 제공
* 전체 서비스의 안정성 대폭 향상

---

## 실전 사례 2: **Azure API Management + Azure Functions 연동**

### 시나리오

한 기업이 Azure API Management를 통해 여러 백엔드 Azure Functions를 호출하는 구조로 RESTful API 서비스를 운영.

### 문제

특정 Function이 부하 또는 외부 의존성으로 자주 timeout 발생 → 전체 API 속도 저하 및 사용자 불만

### 해결

* API Management에서 **policies**를 사용하여 서킷 브레이커 로직 설정

  * 일정 횟수 실패 시 해당 API 호출 차단
  * fallback 응답으로 사용자에게 안내 메시지 전달
* Azure Monitor와 연결해 failure metrics 기반 자동 알림 설정

### 결과

* 무한 재시도 방지
* 클라우드 요금 과다 청구 방지
* 사용자 경험을 제어 가능한 형태로 유지

---

## 실전 사례 3: **Kubernetes + Istio + Envoy의 Circuit Breaker 적용**

### 시나리오

컨테이너 기반 마이크로서비스 아키텍처에서 Istio 서비스 메쉬를 운영 중인 SaaS 기업

### 문제

일부 서비스 pod가 과부하로 인해 응답 지연 → 다른 서비스까지 cascading failure 발생

### 해결

* **Istio DestinationRule**에 circuitBreaker 설정

  * `maxConnections`, `maxPendingRequests`, `consecutiveErrors` 등을 기반으로 자동 차단
* Envoy proxy 레벨에서 요청 큐잉 및 재시도 제어

### 결과

* pod 단위에서 장애 격리
* 전체 클러스터 안정성 향상
* DevOps팀의 대응 시간 단축

---

## 클라우드 구현 팁

| 플랫폼        | 구현 방법                                                                                                   |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| AWS        | Lambda + API Gateway → [AWS Resilience Hub](https://aws.amazon.com/resilience-hub/) 또는 SDK 내 서킷 브레이커 구현 |
| Azure      | API Management policy 또는 Polly (.NET용 라이브러리)                                                            |
| GCP        | Cloud Run + Cloud Load Balancer 설정 또는 Istio 활용                                                          |
| Kubernetes | Istio / Linkerd / Envoy 기반 서비스 메쉬 정책                                                                    |

---

필요하다면, 이와 관련된 **실습 과제** 또는 **시나리오 기반의 문제 해결 과제**도 설계해 드릴 수 있습니다. 원하시면 알려주세요.
