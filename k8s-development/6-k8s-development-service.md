# Kubernetes development: Service

|||
|---|---|
| Title | K8 Development Service |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-development-training-service](https://muellermh.wordpress.com/k8s-development-training-service) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services](https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services) |
| Description | Diese Lektion vermittelt, wie eine Anwendung einen anderen Service innerhalb des Clusters findet: per DNS, per Environment Variable und über Headless Services. |

## Service Discovery

Was ein Service ist, welche Typen es gibt und wie er als YAML beschrieben wird, behandelt bereits [k8s-basic Kapitel 09 Service](../k8s-basic/9-k8s-basic-service.md). Aus Anwendungssicht bleibt die Frage: Wie findet ein Container zur Laufzeit heraus, unter welcher Adresse ein anderer Service erreichbar ist? Kubernetes bietet dafür zwei Mechanismen.

## DNS-basierte Discovery

Der Standardweg ist DNS. Läuft `CoreDNS` im Cluster, bekommt jeder Service automatisch einen DNS-Namen nach folgendem Schema:

```text
<service-name>.<namespace>.svc.cluster.local
```

Innerhalb desselben Namespace reicht dabei bereits der kurze Name `<service-name>`, aus einem anderen Namespace muss mindestens `<service-name>.<namespace>` angegeben werden:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-discovery-demo
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: BACKEND_URL
      value: "http://backend-service.my-namespace.svc.cluster.local:80"
```

Der DNS-Name ist stabil, unabhängig davon, wie oft sich die dahinterliegenden Pods und ihre IPs ändern. Das macht DNS-Discovery zum bevorzugten Ansatz für alles, was zur Laufzeit konfigurierbar sein soll.

## Environment-Variable-basierte Discovery

Der zweite, ältere Mechanismus setzt keine funktionierende DNS-Auflösung voraus. Für jeden Service, der zum Zeitpunkt der Pod-Erstellung bereits existiert, injiziert das Kubelet automatisch Environment Variablen nach diesem Muster in alle neu gestarteten Pods desselben Namespace:

```text
{SVCNAME}_SERVICE_HOST
{SVCNAME}_SERVICE_PORT
```

Für einen Service namens `backend-service` stehen im Container also `BACKEND_SERVICE_SERVICE_HOST` und `BACKEND_SERVICE_SERVICE_PORT` zur Verfügung.

> **Reihenfolge-Falle:** Diese Variablen werden nur für Services gesetzt, die bereits existierten, **bevor** der Pod erstellt wurde. Wird ein Service erst nach dem Pod angelegt, bekommt der bereits laufende Pod die zugehörigen Env-Variablen nicht nachträglich injiziert — dazu müsste der Pod neu erstellt werden. In der Praxis ist DNS-Discovery deshalb robuster, da sie diese Start-Reihenfolge nicht voraussetzt.

## Headless Services

Manche Anwendungen brauchen nicht die gebündelte ClusterIP eines Service, sondern wollen jeden dahinterliegenden Pod einzeln über DNS ansprechen können, etwa um in einem StatefulSet gezielt einen bestimmten Replica zu erreichen. Dafür wird `clusterIP: None` gesetzt:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: my-statefulset-app
  ports:
  - port: 80
```

Ohne ClusterIP löst eine DNS-Abfrage auf `headless-service.my-namespace.svc.cluster.local` nicht mehr auf eine einzelne, gebündelte IP auf, sondern liefert direkt die IP-Adressen aller passenden Pods zurück (bei StatefulSets zusätzlich mit stabilen, pro-Pod eindeutigen DNS-Namen). Der Client entscheidet damit selbst, welchen konkreten Pod er anspricht, statt sich auf das Load-Balancing des Service zu verlassen.
