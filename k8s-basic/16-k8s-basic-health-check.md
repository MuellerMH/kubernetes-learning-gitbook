# Kubernetes basics: Health Check

|||
|---|---|
| Title | K8 Basic Health Checks |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description | Diese Lektion vermittelt, warum Health Checks nötig sind und wie liveness-, readiness- und startupProbe im Zusammenspiel für Selbstheilung und Traffic-Steuerung sorgen. |

## Health Check

Ein Container, der zwar noch läuft aber nicht mehr richtig funktioniert, ist für Kubernetes ohne Health Check unsichtbar. Erst regelmäßige Proben geben dem Kubelet die Grundlage, um zwei Dinge zu tun: verklemmte Container automatisch neu zu starten (Selbstheilung) und Container, die gerade nicht antwortfähig sind, aus der Traffic-Verteilung eines Service zu nehmen (Traffic-Steuerung). Was eine Probe technisch ist und welche Handler-Typen dabei zur Auswahl stehen, behandelt bereits [Kapitel 05 Pod](5-k8s-basic-pod.md) im Abschnitt „Container Proben" — hier geht es darum, wofür Kubernetes diese Proben konkret einsetzt.

## livenessProbe und readinessProbe

Die `livenessProbe` prüft, ob ein Container noch arbeitet. Schlägt sie fehl, gilt der Container als verklemmt und wird vom Kubelet neu gestartet. Die `readinessProbe` prüft dagegen, ob ein Container gerade Anfragen beantworten kann. Schlägt sie fehl, wird der Container nicht neu gestartet, sondern lediglich aus den Endpoints des zugehörigen Service entfernt, bis er wieder erfolgreich antwortet. Beide werden im Pod auf Container-Ebene konfiguriert:

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

Wie die Zeitparameter wie `initialDelaySeconds`, `periodSeconds` oder `failureThreshold` im Detail funktionieren und aufeinander abgestimmt werden, vertieft [k8s-development Kapitel 05 Observability](../k8s-development/5-k8s-development-observability.md).

## startupProbe

Die `startupProbe` prüft, ob die Anwendung im Container überhaupt erfolgreich gestartet ist. Solange sie noch kein einziges Mal erfolgreich war, sind liveness- und readinessProbe für diesen Container deaktiviert — Kubernetes verhält sich so, als gäbe es sie nicht.

Damit löst die startupProbe ein Problem, das eine reine Liveness-Konfiguration nicht lösen kann: Container mit langer Initialisierungsphase, zum Beispiel ein Datenimport beim ersten Start oder ein langsam aufwärmender Cache, brauchen beim Hochfahren oft deutlich mehr Zeit als im späteren Normalbetrieb. Ist die livenessProbe großzügig genug eingestellt, um diese Startzeit zu überstehen, reagiert sie im Normalbetrieb entsprechend träge auf einen tatsächlich verklemmten Container. Die startupProbe trennt beide Phasen voneinander: Erst wenn sie erfolgreich durchgelaufen ist, übernehmen liveness- und readinessProbe wieder ihre reguläre Prüfung. Bis dahin verhindert sie, dass das Kubelet einen Container vorzeitig killt, nur weil er für seinen Start länger braucht, als es für den Normalbetrieb sinnvoll wäre.

## Beispiel

Ein Container mit allen drei Probe-Typen:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
```

Tuning-Parameter, Debugging und das Zusammenspiel mit Logging und Monitoring vertieft [k8s-development Kapitel 05 Observability](../k8s-development/5-k8s-development-observability.md).