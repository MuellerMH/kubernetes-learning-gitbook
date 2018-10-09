# Kubernetes basics: Services

|||
|---|---|
| Title | K8 Basic Services |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Service |

## Service

Ein Kubernetes Service ist eine Abstraction welche die Logik von Pods und Regeln definiert. Die Pods können anhand von [Labels](10-k8-basic-label.md) selektiert werden.

Da Pods jederzeit gestopped und irgendwo anders neu gestarted werden können oder sich die Anzahl der Pods jederzeit ändern kann bietet die Service Abstraction das Routing zu den laufenden Pods, so dass man sich nicht ständig damit beschäftigen muss welche IP Adresse ein Pod gerade hat und wer aktuell auf diese Pods zugriff hat.

## Service beschreiben

Ein Service ist ebenso ein REST Object und kann einfach über eine Yaml Beschreibung definiert werden. Eine beispielhafte Beschreibung des Objektes wäre:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: test-service
  name: test-service
spec:
  ports:
  - name: 80-8080
    port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: test-deployment
  type: NodePort
```

Dieser Service ist nun dafür verantwortlich, dass die Pods mit dem Label `app: test-deployment` innerhalb des Clusters unter dem Port `80` erreichbar ist. Gleichzeitig wird der targetPort des Pots auf die `8080` gemapped, da diese Beispielanwendung innerhalb des Pods auf diesem Port läuft. Zudem sind die Pods `test-deployment` nur über TCP erreichbar.

Kubernetes unterstützt folgende Protokolle:

* TCP (*default)
* UDP
* SCTP ( alpha v1.12)

## Services ohne Selector

In der Regel werden in Service, Pods selektiert um dann den Traffik entsprechend zu routen. In manchen Fällen möchte man jedoch andere Senarios abbilden.

### Senario Beispiel

Sie haben normalerweiße ein externe Produktive Datenbank, möchten aber zum testen eine andere Datenbank nutzten.
Hierzu kannst du einfach ein Service definieren der den Port auf eine definierte IP Adresse leitet.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

Nun kann passend ein Endpunkt definiert werden:

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 1.2.3.4
    ports:
      - port: 9376
```

> Wichtig: die Endpunkte unterstützen keine IPs in:
> * 127.0.0.0/8 (Loopback)
> * 169.254.0.0/16 (link-local)
> * 224.0.0.0/24 (link-local-multicast)
> und können auch keine Cluster IPs sein da die [Kube-Proxy](3-k8s-basic-node.md#kube-proxy) Komponente dies nicht unterstützt.

Der Service funktioniert nun wie ein Service mit Selektoren und leitet den Traffik an den definierten Endpunkt.

## Multi Port Service

Natürlich können auch mehrere Ports in einem Service definiert werden.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9377
```

## Service Types

Kubernetes Service bietet verschiedene Arten der veröffentlichung von Pods um diese zu erreichen.

### ClusterIP

Die ClusterIP ist der Standard, hier wird dem Service eine interne Cluster ip zugeordnet und ist damit innerhalb des Clusters verfügbar.

### NodePort

Der NodePort veröffentlicht einen festen Port für den Service auf dem [Node](3-k8s-basic-node.md). Dadurch ist der Service von außen erreichbar mit der Node Ip und dem Port.

### LoadBalancer

Der LoadBalancer stellt die veröffentlichung des Services über die vom cloud provider zur verfügung gestellte Load Balancer implementierung zur Verfügung.

### ExternalName

ExternalName stellt ein CName eintrag für den Service zur verfügung der in der Service Beschreibung unter externalName definiert wurde.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  type: ExternalName
  externalName: foo.bar.example.com
```

> einem Service können auch externe IP Adressen zugeordnet werden.
> ```yaml
>  spec:
>    externalIPs:
>     - 80.11.12.10
> ```
