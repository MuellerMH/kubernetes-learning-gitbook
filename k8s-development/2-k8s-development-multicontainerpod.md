# Kubernetes development: Multi container pod

|||
|---|---|
| Title | K8 Development Multi Container Pod |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-development-training-multicontainerpod](https://muellermh.wordpress.com/k8s-development-training-multicontainerpod) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers) |
| Description | Diese Lektion vermittelt, wann mehrere Container in denselben Pod gehören und welche etablierten Muster sich dafür durchgesetzt haben. |

## Design

Alle Container eines Pods teilen sich den Netzwerk-Namespace und können optional Volumes gemeinsam nutzen (siehe [Kapitel 01 Core Concept](1-k8s-development-coreconcept.md)). Zwei Container im selben Pod erreichen sich deshalb über `localhost` statt über eine Service-IP, und ein gemeinsam gemountetes `emptyDir`-Volume erlaubt Datenaustausch über das Dateisystem, ohne dass eine der beiden Seiten einen Netzwerkdienst dafür anbieten muss.

Das ist genau dann sinnvoll, wenn ein Hilfsprozess untrennbar zum Lebenszyklus des Hauptcontainers gehört: er wird mit ihm zusammen deployed, skaliert und beendet, und ergibt ohne den Hauptcontainer keinen eigenständigen Sinn. Ein Beispiel ist ein Container, der Logs aus einer Datei ausliest und weiterleitet, während der Hauptcontainer nur in diese Datei schreibt.

Bewusst **nicht** in denselben Pod gehören Komponenten, die unabhängig skalieren müssen oder eine eigene Lebensdauer haben. Frontend und Backend, oder eine Anwendung und ihre Datenbank, laufen deshalb in der Regel in getrennten Pods mit eigenen Deployments und kommunizieren über einen [Service](../k8s-basic/9-k8s-basic-service.md). Landen zu viele fachlich unabhängige Prozesse im selben Pod, verliert die Skalierung ihre Feinkörnigkeit: ein Replica des Pods dupliziert dann immer alle enthaltenen Container gemeinsam, selbst wenn nur einer davon unter Last steht.

## Pattern

Für Hilfscontainer, die dauerhaft neben dem Hauptcontainer laufen, haben sich drei wiederkehrende Muster etabliert:

* **Sidecar** — erweitert den Hauptcontainer um eine Zusatzfunktion, ohne dass der Hauptcontainer davon weiß. Typisches Beispiel ist ein Logging-Sidecar, der die vom Hauptcontainer in ein geteiltes Volume geschriebene Log-Datei ausliest und an ein zentrales Logging-System weiterreicht.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-logging-demo
spec:
  containers:
  - name: app
    image: my-app:1.0
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
  - name: log-shipper
    image: fluent-bit
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
      readOnly: true
  volumes:
  - name: logs
    emptyDir: {}
```

* **Ambassador** — tritt als lokaler Proxy zwischen dem Hauptcontainer und der Außenwelt auf. Der Hauptcontainer spricht einfach mit `localhost`, der Ambassador-Container übernimmt Routing, Retries oder TLS zu einem externen Ziel und kapselt so die Netzwerk-Komplexität vom Hauptcontainer ab.
* **Adapter** — normalisiert die Ausgabe des Hauptcontainers auf ein einheitliches Format, zum Beispiel um uneinheitliche Metrik- oder Log-Formate mehrerer Anwendungen für ein zentrales Monitoring-System anzugleichen.

Alle drei Muster teilen dieselbe Eigenschaft: der Hilfscontainer läuft dauerhaft parallel zum Hauptcontainer. Muss ein Container dagegen nur einmalig **vor** dem Start des Hauptcontainers durchlaufen und terminiert sein, zum Beispiel um Konfiguration vorzubereiten oder auf eine Abhängigkeit zu warten, ist das kein Sidecar-Pattern mehr, sondern ein Init-Container. Dazu mehr in [Kapitel 01 Core Concept](1-k8s-development-coreconcept.md).
