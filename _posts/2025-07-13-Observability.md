---
title: "Observability"
categories: [kubernetes]
---

## Open Telemetry
OpenTelemetry는 애플리케이션을 트레이싱(Trace), 메트릭(Metrics), 로그(Log) 데이터를 표준화된 방식으로 수집하는 오픈소스 프레임워크입니다.
CNCF(Cloud Native Computing Foundation) 프로젝트
다양한 언어 지원 (Java, Python, Go, Node.js 등)
다양한 백엔드로 내보내기 가능 (예: Prometheus, Jaeger, Zipkin 등)

- Logs : 애플리케이션 및 시스템의 이벤트, 상태, 오류 등을 기록한 데이터입니다. 로그는 문제 해결 및 성능 모니터링에 유용합니다.
- Metrics : 시스템의 성능 및 상태를 수치로 표현한 데이터입니다. 예를 들어, CPU 사용률, 메모리 사용량, 요청 처리 시간 등이 있습니다.
시간에 따라 변화하는 데이터를 수집하여 시스템의 성능을 모니터링하고, 경향을 분석하는 데 사용됩니다.
  - Gauge : 특정 시점의 값을 나타내는 메트릭입니다. 예를 들어, 현재 CPU 사용률, 메모리 사용량 등이 있습니다.
  - Counter : 증가하는 값으로, 이벤트의 발생 횟수를 나타냅니다. 예를 들어, 요청 수, 오류 수 등이 있습니다.
  - Meter : 시간에 따른 이벤트 발생률을 나타내는 메트릭입니다. 예를 들어, 초당 요청 수, 초당 오류 수 등이 있습니다.심장 박동 메트릭이라고도 하며, 시스템의 상태를 주기적으로 확인하는 데 사용됩니다.
  - Histogram : 값의 분포를 나타내는 메트릭입니다. 예를 들어, 요청 처리 시간의 분포, 응답 크기의 분포 등이 있습니다. 히스토그램은 데이터의 분포와 경향을 분석하는 데 유용합니다. 
    - bucket : 히스토그램에서 특정 범위의 값을 나타내는 구간입니다. 예를 들어, 요청 처리 시간이 0-100ms, 100-200ms, 200-300ms 등으로 나뉘어 있을 수 있습니다.
    - bin : 히스토그램에서 bucket의 개수를 나타내는 단위입니다. 예를 들어, 10개의 bucket이 있다면, bin은 10이 됩니다.
    - count : 히스토그램에서 각 bucket에 해당하는 값의 개수를 나타냅니다. 예를 들어, 0-100ms bucket에 50개의 요청이 있었다면, count는 50이 됩니다.
- Traces : 분산 시스템에서 요청의 흐름을 추적한 데이터입니다. 트레이스는 요청이 시스템을 통과하는 경로를 시각화하여 성능 병목 현상을 식별하는 데 도움을 줍니다.
- Alerts : 시스템의 상태나 성능이 특정 기준을 초과하거나 미달할 때 발생하는 경고입니다. 알림은 문제를 조기에 감지하고 대응할 수 있도록 도와줍니다.

## kube-prometheus
`curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash`
`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`
`helm install my-observability prometheus-community/kube-prometheus-stack --version 55.5.0`
Prometheus는 시스템과 애플리케이션에서 수집한 시계열(Time-series) 메트릭 데이터를 저장하고, 쿼리하고, 경고(Alerts)를 생성합니다.

kube-prometheus는 Kubernetes 클러스터의 모니터링을 위한 오픈 소스 솔루션입니다. Prometheus, Grafana, Alertmanager 등을 포함하여 Kubernetes 클러스터의 상태와 성능을 모니터링하고 시각화하는 데 사용됩니다.

- kube-prom-operator : Prometheus 및 Grafana와 같은 모니터링 도구를 Kubernetes 클러스터에서 관리하고 배포하는 데 사용되는 오픈 소스 프로젝트입니다. kube-prometheus는 Prometheus Operator를 기반으로 하며, Kubernetes 리소스를 사용하여 Prometheus 및 Grafana 인스턴스를 쉽게 배포하고 관리할 수 있습니다.
- Alert Manager : Prometheus와 함께 사용되는 경고 관리 도구로, 메트릭 데이터를 기반으로 경고를 생성하고, 이를 이메일, Slack, PagerDuty 등 다양한 채널로 전송할 수 있습니다. Alertmanager는 경고의 그룹화, 라우팅 및 이력 관리를 지원합니다.
- Node Exporter : Prometheus와 함께 사용되는 에이전트로, 노드의 메트릭 데이터를 수집하여 Prometheus에 전송합니다. Node Exporter는 CPU, 메모리, 디스크, 네트워크 등의 시스템 리소스 사용량을 모니터링하는 데 사용됩니다.(daemonset으로 배포)
- Kube-state-metrics : Kubernetes 클러스터의 상태 정보를 수집하여 Prometheus에 전송하는 에이전트입니다. Kube-state-metrics는 Pod, Deployment, Service 등의 Kubernetes 리소스 상태를 모니터링하는 데 사용됩니다.(deployment로 배포)
- Prometheus Adapter : Kubernetes 클러스터의 메트릭 데이터를 Prometheus에서 수집하여 Metric화.
- Grafana Operator : Grafana 대시보드를 Kubernetes 클러스터에서 관리하고 배포하는 데 사용되는 오픈 소스 프로젝트입니다. Grafana Operator는 Grafana 인스턴스를 쉽게 배포하고, 대시보드를 관리하며, 알림을 설정할 수 있도록 지원합니다.






## Grafana
Grafana는 오픈 소스 데이터 시각화 및 모니터링 도구로, 다양한 데이터 소스(예: Prometheus, InfluxDB, Elasticsearch 등)에서 데이터를 시각화하고 대시보드를 생성하는 데 사용됩니다. Grafana는 사용자 친화적인 인터페이스를 제공하여 데이터를 쉽게 시각화하고, 대시보드를 공유하며, 알림을 설정할 수 있습니다.

[App] 
  ↓
(OpenTelemetry: 트레이스, 메트릭 수집)
  ↓
[Prometheus: 메트릭 저장 및 쿼리]
  ↓
[Grafana: 시각화 및 알람]