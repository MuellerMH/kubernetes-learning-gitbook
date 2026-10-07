# Kubernetes Advance Training Intro

|||
|---|---|
| Title | K8 Advance training |
| Category | Course |
| Level | proficient |
| Duration | ? |
| Url | ... |
| Description | By the end of this course you will understand the advance topics of k8s. You will learn how discovery,  ingress, volumes, deamons, resources and autoscalling works.   |

## Was dich in diesem Kurs erwartet

Die Advance-Sektion baut direkt auf den [Basics](../k8s-basic/0-k8s-basic-intro.md) auf. Du solltest Pods, Deployments, Services, Labels und Secrets sicher beherrschen, bevor du hier einsteigst.

- [01 Discovery](1-k8s-advance-discovery.md) — wie Pods Services über Environment Variablen und DNS finden
- [02 ConfigMap](2-k8s-advance-config-map.md) — Konfiguration vom Container Image trennen
- [03 Ingress](3-k8s-advance-ingress.md) — HTTP(S)-Routing von außen in den Cluster
- [04 Volumes](4-k8s-advance-volumes.md) — PersistentVolumes, PersistentVolumeClaims und StorageClasses
- [05 DaemonSet](5-k8s-advance-deamon.md) — node-gebundene Workloads, Tolerations und Update-Strategien
- [06 Resources](6-k8s-advance-resources.md) — Requests, Limits und Quality of Service Klassen
- [07 Autoscaling](7-k8s-advance-autoscalling.md) — Pods automatisch mit dem HorizontalPodAutoscaler skalieren

## Voraussetzungen

- Ein laufender Test-Cluster, zum Beispiel mit [kind](https://kind.sigs.k8s.io/), [minikube](https://minikube.sigs.k8s.io/) oder [k3d](https://k3d.io/)
- `kubectl` ist installiert und auf den Cluster konfiguriert (`kubectl cluster-info` liefert eine Antwort)
- Für das Autoscaling-Kapitel: ein installierter Metrics Server (`minikube addons enable metrics-server`)
- Für das Ingress-Kapitel: ein installierter Ingress Controller (`minikube addons enable ingress`)
