# Kubernetes Development Training Intro

|||
|---|---|
| Title | K8 Development training |
| Category | Course |
| Level | proficient |
| Duration | ? |
| Url | ... |
| Description | Diese Sektion richtet sich an Entwickler, die Anwendungen für Kubernetes bauen: Pod-Lifecycle und Container-Konfiguration, Multi-Container-Pattern, Pod-Design mit Deployments/Jobs/CronJobs, Security-Context/Service-Accounts, Observability (Probes, Logging, Debugging) sowie Service-Discovery und Netzwerksicht aus Anwendungsperspektive. |

## Was dich in diesem Kurs erwartet

Die Inhalte orientieren sich an den Themen der Certified Kubernetes Application Developer (CKAD) Prüfung.

- [01 Core Concept](1-k8s-development-coreconcept.md) — Pod-Lifecycle, `command`/`args`, Environment Variablen, Resources und Security Context
- [02 Multi Container Pod](2-k8s-development-multicontainerpod.md) — Sidecar, Ambassador und Adapter Pattern
- [03 Pod Design](3-k8s-development-poddesign.md) — Labels, Deployments, Rolling Updates, Jobs und CronJobs
- [04 Configuration](4-k8s-development-configuration.md) — ConfigMaps, Secrets, SecurityContext und ServiceAccounts
- [05 Observability](5-k8s-development-observability.md) — Probes, Logging, Monitoring und Debugging
- [06 Service](6-k8s-development-service.md) — Services aus Anwendungssicht
- [07 Networking](7-k8s-development-networking.md) — DNS, Service Discovery und NetworkPolicies für Entwickler

## Voraussetzungen

- Die Inhalte der [Basics](../k8s-basic/0-k8s-basic-intro.md) sollten bekannt sein.
- Grundkenntnisse in Docker bzw. Container Images (Image bauen, taggen, in eine Registry pushen).
- Ein lokaler Test-Cluster (kind, minikube oder k3d) und `kubectl`.

## Tipps für die CKAD-Prüfung

- Zeit ist der knappste Faktor. Lege dir Aliase an, zum Beispiel `alias k=kubectl` und `export do="--dry-run=client -o yaml"`, und erzeuge YAML-Vorlagen mit `k run nginx --image=nginx $do > pod.yaml`.
- Nutze `kubectl explain pod.spec.containers` statt das Schema auswendig zu lernen.
- Prüfe nach jeder Aufgabe das Ergebnis mit `kubectl get`, `kubectl describe` und `kubectl logs`.
