# Kubernetes 플랫폼 서비스 구성 리포지토리

## 1. 프로젝트 개요

이 리포지토리는 Kubernetes 클러스터의 핵심 플랫폼 서비스를 관리하기 위한 Helm 차트들을 포함한다.

주요 목적은 GitOps 원칙에 따라 플랫폼 서비스(특히 모니터링 스택)의 구성을 버전 관리하고, 일관되게 배포하는 것이다. 현재 이 리포지토리는 `kube-prometheus-stack`을 기반으로 한 포괄적인 모니터링 솔루션을 중점적으로 관리한다.

## 2. 핵심 구성 요소: Kube-Prometheus-Stack

이 리포지토리의 핵심은 `monitoring/prometheus-stack` 디렉토리에 위치한 커스터마이징된 **`kube-prometheus-stack` Helm 차트**다. 이 차트는 다음을 포함하는 올인원(All-in-one) 모니터링 솔루션을 제공한다:

-   **Prometheus**: 메트릭 수집 및 시계열 데이터베이스.
-   **Grafana**: 메트릭 시각화를 위한 대시보드.
-   **Alertmanager**: 알림(Alerting) 규칙 관리 및 발송.
-   **Prometheus Operator**: `ServiceMonitor`, `PodMonitor`, `PrometheusRule`과 같은 CRD를 통해 Prometheus 설정을 Kubernetes 네이티브 방식으로 자동화.
-   **Node Exporter & Kube State Metrics**: 노드 및 Kubernetes 객체 상태에 대한 메트릭 수집.

## 3. 디렉토리 구조

```
.
└── monitoring/
    └── prometheus-stack/   # kube-prometheus-stack Helm 차트
        ├── Chart.yaml      # 차트 정보
        ├── values.yaml     # 차트의 모든 설정을 제어하는 핵심 파일
        ├── templates/      # Prometheus, Grafana, Alertmanager 등 모든 리소스 템플릿
        ├── charts/         # 의존성을 가지는 하위 차트 (Sub-charts)
        └── etc-app/        # 추가적인 커스텀 ServiceMonitor 등 (예: ArgoCD 모니터링)
```

-   **`monitoring/prometheus-stack/`**: 이 리포지토리의 핵심인 Helm 차트 디렉토리.
-   **`values.yaml`**: 모니터링 스택의 모든 세부 설정(리소스 할당, Grafana 대시보드, Prometheus 규칙 등)을 이 파일에서 제어한다. **이 파일이 이 모니터링 스택의 "Source of Truth"다.**
-   **`templates/`**: `kube-prometheus-stack`의 기본 템플릿과 커스텀 템플릿을 포함한다.
-   **`etc-app/`**: 기본 차트 외에 추가적으로 모니터링할 대상을 위한 `ServiceMonitor`와 같은 커스텀 매니페스트를 관리한다.

## 4. 배포 전략 (GitOps)

> **주의:** 이 리포지토리의 Helm 차트는 `helm install` 명령어로 직접 설치하기보다, GitOps 워크플로우를 통해 배포되도록 설계되었다.

배포는 `k8s-gitops-configs` 리포지토리에 정의된 Argo CD `Application` 매니페스트에 의해 관리된다.

**워크플로우:**
1.  `k8s-gitops-configs` 리포지토리의 `argocd-apps/monitoring/monitoring-stack.yaml` 파일은 Argo CD 애플리케이션을 정의한다.
2.  이 Argo CD 애플리케이션은 `source` 필드를 통해 이 `k8s-platform-configs` 리포지토리의 `monitoring/prometheus-stack` 경로를 바라본다.
3.  Argo CD는 해당 경로의 Helm 차트를 감지하고, `values.yaml` 파일의 내용을 기반으로 모든 Kubernetes 리소스를 렌더링하여 클러스터에 배포 및 동기화한다.

## 5. 모니터링 스택 커스터마이징

모니터링 스택의 설정을 변경하려면, 이 리포지토리의 `monitoring/prometheus-stack/values.yaml` 파일을 수정하고 Git에 푸시하면 된다. Argo CD가 변경 사항을 감지하여 자동으로 클러스터에 적용할 것이다.

예를 들어, Grafana의 리소스 할당량을 늘리거나 새로운 Prometheus 알림 규칙을 추가하는 등의 모든 변경은 이 `values.yaml` 파일을 통해 이루어져야 한다.
