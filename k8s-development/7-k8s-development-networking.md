# Kubernetes development: Networking

|||
|---|---|
| Title | K8 Development Networking |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-development-training-networking](https://muellermh.wordpress.com/k8s-development-training-networking) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/) |
| Description | Diese Lektion vermittelt die Netzwerk-Sicht aus Anwendungssicht: Container-interne Kommunikation und die NetworkPolicy, die die eigene Anwendung mitliefern sollte. |

Diese Lektion behandelt Netzwerk-Themen bewusst aus Anwendungsperspektive. Die Cluster-Infrastruktursicht — CNI-Plugin, kube-proxy, CoreDNS und die Mechanik von NetworkPolicy — behandelt bereits [k8s-admin Kapitel 03 Network](../k8s-admin/3-k8s-admin-network.md), inklusive eines vollständigen YAML-Beispiels.

## Basics

Wie in [Kapitel 02 Multi Container Pod](2-k8s-development-multicontainerpod.md) beschrieben, teilen sich alle Container innerhalb eines Pods denselben Netzwerk-Namespace, dieselbe IP und denselben Port-Space. Container im selben Pod erreichen sich deshalb immer über `localhost`, nie über eine Service- oder Pod-IP — das gilt unabhängig davon, ob der jeweils andere Container über einen Service nach außen erreichbar ist oder nicht.

Ein Detail, das in der Praxis häufig zu Verwirrung führt: `containerPort` in der Pod-Spezifikation ist rein informativ. Es deklariert, auf welchem Port die Anwendung im Container voraussichtlich lauscht, verhindert aber nicht, dass die Anwendung tatsächlich auf einem anderen Port lauscht, und öffnet auch keinen Port, der nicht ohnehin schon offen wäre.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: networking-demo
spec:
  containers:
  - name: app
    image: my-app:1.0
    ports:
    - containerPort: 8080
```

Entscheidend ist, dass der Wert in `containerPort` mit dem tatsächlichen Listening-Port der Anwendung übereinstimmt, da nachgelagerte Objekte wie ein Service über `targetPort` genau diesen Port ansprechen. Weicht der reale Listening-Port vom deklarierten `containerPort` ab, bleibt der Container über den Service unerreichbar, obwohl Kubernetes selbst keinen Fehler meldet.

## Network policies

Wie eine `NetworkPolicy` syntaktisch aufgebaut ist und was `podSelector`, `Ingress` und `Egress` bedeuten, behandelt bereits [k8s-admin Kapitel 03 Network](../k8s-admin/3-k8s-admin-network.md) mit einem vollständigen Beispiel. Aus Entwicklersicht zählt vor allem eine Frage: welche NetworkPolicy muss die eigene Anwendung mitbringen, damit sie im Cluster funktioniert, ohne mehr Traffic zu erlauben als nötig?

Konkret bedeutet das, für die eigene Anwendung zu benennen:

* welche **Ingress-Quellen** tatsächlich mit ihr sprechen müssen (z. B. nur der eigene Frontend-Pod, nicht der gesamte Namespace)
* welche **Egress-Ziele** sie selbst ansprechen muss (z. B. eine bestimmte Datenbank, ein externer API-Endpunkt, DNS)

Ein bewährtes Ausgangsmuster ist Default-Deny: zuerst wird jeglicher Traffic zu den eigenen Pods gesperrt, danach werden nur die tatsächlich benötigten Verbindungen gezielt wieder erlaubt. Das verhindert, dass eine Anwendung stillschweigend mehr Traffic annimmt oder erzeugt, als sie fachlich braucht.

Ob eine gesetzte NetworkPolicy tatsächlich das gewünschte Verhalten zeigt, lässt sich direkt aus einem Debug-Pod heraus testen:

```shell
kubectl run netpolicy-test --rm -it --image=busybox --restart=Never -- \
  wget -qO- --timeout=2 http://backend-service.my-namespace.svc.cluster.local:80
```

Schlägt der Request von einem Pod fehl, der laut Policy keinen Zugriff haben soll, während er von einem erlaubten Pod aus funktioniert, greift die Policy wie beabsichtigt. Die allgemeine Debugging-Vorgehensweise für Services und Verbindungstests behandelt [Kapitel 06 Observability](5-k8s-development-observability.md).
