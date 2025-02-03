# Open-Telemetry 톺아보기

최근, 사내 Observability 개선을 위한 Session을 진행했다.

세션의 내용은 "왜 Observability가 중요한가?" 에 대한 내용이었는데,
그 게시글에서 중요히 다뤘던 Open-Telemetry는 무엇인가? 어떻게 해야하는가? 에 대해서는 시간 관계 상 전달할 수 없었다.

짧은 시간 내에 목적을 전달하자니, 내가 설명하고싶었던 모든 것을 전달할 수 없었다.

이번 게시글은 전달하지 못한 내용들을 정리해서,
추후 비슷한 이야기를 할 필요가 있을 경우 이 게시글을 전달할까 한다.
---

# Open-Telemetry : 정의

흔히 O-Tel이라고 줄여 부르는 Open-Telemetry는 Observability 데이터를 수집하고 전송하는 오픈소스 프로젝트다.

Cloud Native Computing Foundation(CNCF)에서 관리하며, 다양한 언어와 환경에서 사용할 수 있다.

간단히 말해, Open-Telemetry는 Telemetry 업계의 표준을 정의하려고 만든 프레임워크라고 할 수 있다.

업계 표준이라고 하는 게 빈말은 아닌게, 실제로 대부분의 벤더에서 O-Tel을 사용하여 연동할 수 있다.

또한 많은 사용을 도모하기 위함인지, 여러 언어 및 프레임워크에서 O-Tel을 사용하기 위한 라이브러리나 SDK 등 쉬운 연동이 가능하게끔 구성되어 있다.

O-Tel과 기존 모니터링 도구들의 agent를 어느정도 비교하자면 아래 표와 같다.

| 비교 항목  | Open-Telemetry             | 기존 APM 도구 (New Relic, Datadog 등) |
|--------|----------------------------|----------------------------------|
| 표준화    | ✅ OTel 표준 제공               | ❌ 각 벤더마다 데이터 형식 다름               |
| 벤더 종속성 | ✅ 없음                       | ❌ 특정 벤더 종속                       |
| 확장성    | ✅ 다양한 언어 및 환경 지원           | ⭕ 일부 제한 있음                       |
| 데이터 통합 | ✅ Metrics, Logs, Traces 통합 | ❌ 대부분 별도 관리                      |
| 비용     | ✅ 오픈소스 무료                  | ❌ 대부분 유료                         |

# Open-Telemetry : 구성요소

Open-Telemetry는 다음과 같은 주요 구성 요소로 이루어져 있다.

1. API & SDK

- API : Open-Telemetry를 사용하여 데이터를 수집하는 인터페이스
- SDK : API를 기반으로 데이터를 가공하고 전송하는 구현체

2. Collector

- 애플리케이션에서 수집한 데이터를 중앙에서 가공 및 전송하는 역할
- 다양한 모니터링 시스템(AWS X-Ray, Jaeger, Prometheus 등)로 데이터를 보내거나 받아올 수 있음

3. Exporter

- Open-Telemetry 데이터를 다양한 저장소 및 모니터링 툴로 전송하는 모듈
- Prometheus Exporter, Jaeger Exporter, Elasticsearch Exporter 등

일단 개략적으로 보기 위해 정리했는데, 조금 더 자세히 알아보자.
![otel-diagram.svg](img/otel-diagram.svg)
O-Tel은 크게 세 가지 구성요소로 판단할 수 있다.

## 첫 번째로, 모니터링을 하기 위한 '대상'이다.
이는 Application이 될 수도 있고, Database, 혹은 장비 그 자체가 될 수도 있다.

아쉽게도, 장비에 아무런 짓을 하지 않고도 상태를 알 수 있는 방법은 아니지만

각 장비에 맞는 agent를 구성할 경우, 웬만하면 zero-code 상태로 내 Application의 상태를 전달할 수 있다.

## 두 번째로, 모니터링 Application에 전달하기 위한 '중간 관리자'이다.
내 데이터를 여러 모니터링 Application에 전송하려면, 구성이 필요할 뿐더러 모니터링 솔루션이 변경될 경우 모든 Service에 접근하여 수정할 소요가 발생한다.

이를 줄여주고, 내가 모니터링 Application을 '몰라도 괜찮도록' 구성하려면 중간 관리자가 필요하다.

이가 Open-Telemetry Collector이다.

개인적으로 Open-Telemetry Collector가 Open-Telemetry의 핵심 구성요소라고 생각하는데, 이는 따로 Section을 빼서 자세히 설명하겠다.

## 세 번째로, 모니터링 한 데이터를 받아, 관리하기 위한 '목적지' 이다.
데이터를 Collector에서 받음으로써 끝나는 것이 아니라, 데이터를 원하는 모니터링 Application으로 전달해야 한다.

이는 모니터링 Application 또한 O-Tel을 받아들일 준비가 되어 있어야한다는 것인데,

O-Tel에서는 이 역할을 Exporter라고 한다.

사실 내가 개발하는 것은 아니라, 여러 오픈소스에서 이미 활발히 개발되어 있다.

필요하다면 개발하겠지만, 아직 한 번도 개발해본 적은 없다.

# Open-Telemetry Collector
아까 제일 중요하다고 했던 Open-Telemetry Collector이다.

위 단락의 '중간 관리자'에 해당하는 Agent인데

O-Tel Collector를 사용할 경우
- 벤더 종속이 제거되어 application의 자유도가 증가하고
- 여러 source에서 들어오는 데이터를 모아 가공 후 전송할 수 있고
- 데이터 필터, 샘플링, 마스킹 등의 변경을 할 수 있으며
- 배치 전송을 통해 네트워크 대역 이슈 발생률이 줄어들 수 있다.

## Open-Telemetry Collector : 구성요소
O-Tel Collector는 크게 Receiver, Processor, Exporter, Extensions 라는 요소로 구성된다.

### Receiver
Application에서 데이터를 수집하는 역할을 한다.

다양한 Receiver가 이미 제공되고 있으며, 원하는 경우 커스터마이징 또한 가능하다.

### Processor
수집된 데이터를 가공하고 최적화하는 역할을 한다.

자유도가 높은 기능인데, 이 부분은 사용자의 커스터마이징이 가능하다.

간단히 데이터를 모아서 전송하는 기능, 데이터 필터링, 데이터 추가/수정, 데이터 가공 등 여러 부분에서 활용할 수 있다.
### Exporter
가공된 데이터를 최종 목적지로 전송하는 역할을 한다.

각각의 모니터링 Application에 전송하는 방법을 제공할 수 있다.

### Extensions
Collector의 기능을 확장하는 부가적인 모듈이다.

Health-Check, Pprof(Collector 메모리, CPU 모니터링), fileLog 등 구현되어 있지 않을 것 같은 많은 로직들이 위 Section에 해당한다.

이를 그림으로 나타내면 
![otel-collector.svg](img/otel-collector.svg)

대충 이런 느낌이 된다.

개인적으로, Spring Cloud Dataflow라던가, Spring Batch같은 아이들과 비슷하게 받아들일 수 있을 것 같다.

# Open-Telemetry : 적용

[모니터링 Application Docker Compose](https://github.com/adszzz11/monitoring-integration)

개인 Repository에 이전에 작성했던 Docker Compose 파일을 담았다.

현재 최신 버전이 아니므로 동작이 원활히 되지 않을 수 있으나

이런 식으로의 구성에서 벗어나지 않으므로 최신 문서와 레포지토리 상 내용을 보면 쉽게 구성할 수 있을 것이다.

# Cons.
Open-Telemetry를 사용한다는 것은 사실

모니터링 Application이 뭘로 바뀔지 모르니까, 다 대비해줄게! 하는 것과 비슷하다.

JSON처럼, 하나의 약속된 프로토콜 정도라고 생각하고 있다.

실제 사용기나 우여곡절을 겪었으면 더 자세하고 맛있는 글이 되었을텐데, 그건 앞으로 겪어볼 예정이지 않을까 한다.