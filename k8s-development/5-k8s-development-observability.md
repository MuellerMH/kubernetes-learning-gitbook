# Kubernetes development: Observability

|||
|---|---|
| Title | K8 Development Observability |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-development-training-observability](https://muellermh.wordpress.com/k8s-development-training-observability) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) |
| Description | Diese Lektion vermittelt Liveness- und Readiness-Probes, Logging-Konventionen, die Monitoring-Einordnung sowie Debugging von Anwendungen und Services. |

## Liveness probes

Eine Liveness-Probe prüft, ob ein Container noch arbeitsfähig ist. Schlägt sie wiederholt fehl, tötet das Kubelet den Container und startet ihn gemäß `restartPolicy` neu (siehe [Kapitel 01 Core Concept](1-k8s-development-coreconcept.md)). Sie ist damit das Werkzeug gegen verklemmte Prozesse, die zwar noch laufen, aber in einem Zustand hängen, aus dem sie sich selbst nicht mehr befreien.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
  failureThreshold: 3
```

## Readiness probes

Eine Readiness-Probe prüft dagegen, ob ein Container aktuell in der Lage ist, Anfragen zu beantworten. Schlägt sie fehl, wird der Container **nicht** neu gestartet, sondern lediglich aus den Endpoints der zugehörigen Services entfernt — er bekommt so lange keinen Traffic mehr, bis die Probe wieder erfolgreich ist. Das ist der zentrale Unterschied zur Liveness-Probe: Liveness entscheidet über Neustart, Readiness entscheidet über Traffic-Zustellung. Eine Anwendung, die beim Start erst eine Verbindung zur Datenbank aufbauen oder einen Cache füllen muss, sollte das über eine Readiness-Probe abbilden, nicht über Liveness — sonst killt Kubernetes den Container, obwohl er nur noch nicht fertig hochgefahren ist.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3
```

Beide Probe-Typen unterstützen dieselben drei Prüfmechanismen:

Mechanismus | Beschreibung
--- | ---
`httpGet` | ein HTTP-Request gegen Pfad und Port gilt als erfolgreich bei einem Statuscode zwischen 200 und 399
`exec` | ein Befehl wird im Container ausgeführt, Exit-Code `0` gilt als Erfolg
`tcpSocket` | es wird nur geprüft, ob sich eine TCP-Verbindung zum angegebenen Port aufbauen lässt

Zwei Zeitparameter bestimmen, wie tolerant eine Probe ist:

* `initialDelaySeconds` — wie lange das Kubelet nach dem Start des Containers wartet, bevor die erste Probe überhaupt ausgeführt wird. Zu niedrig gesetzt, meldet eine noch langsam startende Anwendung fälschlich Fehler
* `failureThreshold` — wie viele aufeinanderfolgende Fehlschläge nötig sind, bevor die Probe als "failed" gilt und die entsprechende Konsequenz (Neustart bei Liveness, Entfernen aus den Endpoints bei Readiness) eintritt

## Logging

Kubernetes erwartet, dass eine Anwendung ihre Logs nach `stdout`/`stderr` schreibt, statt in eine Datei innerhalb des Containers. Nur so kann das Kubelet die Ausgabe einsammeln und über `kubectl logs` zugänglich machen:

```shell
kubectl logs <pod-name>
kubectl logs -f <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> --previous
```

* `-f` folgt dem Log-Stream fortlaufend, statt nur den aktuellen Stand einmalig auszugeben
* `-c` wählt bei einem Multi-Container-Pod (siehe [Kapitel 02 Multi Container Pod](2-k8s-development-multicontainerpod.md)) gezielt einen Container aus, ansonsten nimmt `kubectl` an, dass der Pod nur einen Container enthält
* `--previous` zeigt die Logs des vorherigen, bereits beendeten Containers — der Standardfall nach einem Absturz, da der Log-Stream des neuen Containers bei null beginnt (siehe `CrashLoopBackOff` in [Kapitel 01 Core Concept](1-k8s-development-coreconcept.md))

## Monitoring

Ob und wie eine Anwendung von einem Prometheus-Server automatisch als Scrape-Ziel erkannt wird, steuern Annotations auf dem Pod (Syntaxbeispiel dazu in [k8s-basic Kapitel 10 Annotations](../k8s-basic/10-k8s-basic-annotations.md)). Fachlich bedeutet das: die eigene Anwendung muss einen Metrik-Endpoint (üblicherweise `/metrics`) im Prometheus-Textformat bereitstellen, damit ein per Annotation konfigurierter Scraper überhaupt etwas findet — die Annotation allein erzeugt keine Metriken.

Für einen schnellen, punktuellen Blick auf den aktuellen Ressourcenverbrauch ohne eigenes Monitoring-System steht `kubectl top` zur Verfügung, sofern der Metrics-Server im Cluster läuft:

```shell
kubectl top pods
kubectl top nodes
```

## Debugging

### Application

Für die Fehlersuche direkt in einem laufenden Container steht `kubectl exec` bereit, um eine Shell oder einen Einzelbefehl im Container auszuführen:

```shell
kubectl exec -it <pod-name> -- /bin/sh
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

Fehlt im Container-Image überhaupt eine Shell oder Debugging-Werkzeuge (z. B. bei minimalen Images wie `distroless`), lässt sich stattdessen ein temporärer Ephemeral Container an den laufenden Pod anhängen:

```shell
kubectl debug -it <pod-name> --image=busybox --target=<container-name>
```

Der Ephemeral Container läuft im selben Pod-Netzwerk-Namespace und kann bei `--target` sogar den Prozess-Namespace des Zielcontainers mitnutzen, ohne dass der eigentliche Container dafür neu gebaut oder neu gestartet werden muss.

Für den Überblick über Zustand und Vorgeschichte eines Pods liefert `kubectl describe` die Events des Pods, inklusive Scheduling-Entscheidungen, Image-Pull-Fehlern und fehlgeschlagenen Probes:

```shell
kubectl describe pod <pod-name>
```

### Services

Bei einer Anwendung, die über einen Service erreichbar sein soll, aber nicht erreichbar ist, lohnt sich die Fehlersuche in dieser Reihenfolge:

1. **Port-Forward zum direkten Test**, unter Umgehung von Service und Ingress:

```shell
kubectl port-forward pod/<pod-name> 8080:8080
```

Antwortet die Anwendung über den Port-Forward, liegt das Problem nicht im Container, sondern in der Service- oder Netzwerk-Konfiguration davor.

2. **Endpoints prüfen** — ein Service ohne passende Endpoints leitet keinen Traffic weiter, meist weil kein Pod das erwartete Label trägt oder keine Readiness-Probe erfolgreich ist:

```shell
kubectl get endpoints <service-name>
```

3. **DNS-Auflösung aus einem Debug-Pod testen**, um zu prüfen, ob der Servicename überhaupt aufgelöst wird:

```shell
kubectl run dns-test --rm -it --image=busybox --restart=Never -- nslookup <service-name>
```

Details zur clusterinternen Service-Discovery per DNS behandelt [Kapitel 07 Service](6-k8s-development-service.md).
