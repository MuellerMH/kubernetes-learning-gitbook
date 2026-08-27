# Kubernetes advance: Discovery

|||
|---|---|
| Title | K8 Advance Discovery |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-advance-training-discovery](https://muellermh.wordpress.com/k8s-advance-training-discovery) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services](https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services)<br>[https://kubernetes.io/docs/concepts/services-networking/service/#endpointslices](https://kubernetes.io/docs/concepts/services-networking/service/#endpointslices)<br>[https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) |
| Description | Diese Lektion vertieft Service Discovery: Endpoints/EndpointSlices, die Kopplung an Readiness, SRV-Records für benannte Ports sowie die Steuerung des Pod-DNS-Verhaltens über dnsPolicy und dnsConfig. |

## Einordnung

Das DNS-Grundschema, die Environment-Variable-Discovery und Headless Services behandelt bereits [k8s-development Kapitel 06 Service](../k8s-development/6-k8s-development-service.md). Dieses Kapitel setzt dort auf und geht auf das, was intern hinter einem Service steckt, sowie auf die feineren Stellschrauben der Discovery ein: Endpoints/EndpointSlices, die Kopplung an Readiness, SRV-Records und die Steuerung des Pod-DNS-Verhaltens.

## Endpoints und EndpointSlices

Ein Service selbst routet keinen Traffic. Er hält lediglich Selector und Ports, während ein separates Objekt die tatsächlichen IP-Adressen der passenden Pods vorhält. Historisch war das ein einzelnes `Endpoints`-Objekt pro Service, in dem alle Adressen gesammelt gelistet wurden.

Laut aktueller Kubernetes-Dokumentation ist die `Endpoints`-API deprecated zugunsten von `EndpointSlices`. Der Grund ist Skalierung: Statt alle Adressen in einem monolithischen `Endpoints`-Objekt zu bündeln, verteilt Kubernetes sie auf mehrere kleinere `EndpointSlice`-Objekte (standardmäßig maximal 100 Endpoints pro Slice). Das lässt sich zusammen mit dem Service abfragen:

```shell
kubectl get endpointslices -l kubernetes.io/service-name=<svc>
```

Jede EndpointSlice referenziert dabei über das Label `kubernetes.io/service-name` den Service, zu dem sie gehört. Bei einem Service mit sehr vielen Pods dahinter zeigt `kubectl get endpointslices` entsprechend mehrere Objekte für denselben Service, während `kubectl get endpoints` weiterhin nur ein einzelnes, gesammeltes Objekt liefert.

## Readiness und Discovery-Kopplung

In Endpoints und EndpointSlices landen nur Pods, deren `readinessProbe` aktuell erfolgreich ist. Ein Pod, der läuft aber nicht ready ist, wird aus der Adressliste entfernt, ohne dass der Pod selbst neu gestartet wird. Discovery und Health Check sind damit direkt gekoppelt: DNS-Auflösung und Environment-Variablen aus [Kapitel 06 Service](../k8s-development/6-k8s-development-service.md) liefern immer nur die Pods, die laut Readiness auch tatsächlich Traffic annehmen können. Was eine Probe technisch ist und wie liveness-, readiness- und startupProbe zusammenspielen, behandelt bereits [k8s-basic Kapitel 16 Health Check](../k8s-basic/16-k8s-basic-health-check.md).

## DNS: SRV-Records für benannte Ports

Hat ein Service benannte Ports (`ports[].name`), stellt CoreDNS dafür zusätzlich SRV-Records bereit. Das Format lautet laut Kubernetes-Dokumentation:

```text
_<port-name>._<protocol>.<service>.<namespace>.svc.cluster.local
```

Bei einem regulären Service löst dieser SRV-Record auf Portnummer und den Domainnamen des Service auf. Bei einem Headless Service liefert derselbe SRV-Record dagegen mehrere Antworten, eine pro dahinterliegendem Pod, jeweils mit dem eigenen Pod-Hostnamen vorangestellt. Ein Client, der einen SRV-Record abfragt, bekommt damit nicht nur die Adresse, sondern auch den konfigurierten Port zurück, ohne ihn hart zu verdrahten.

## Pod-DNS-Verhalten steuern

Über `spec.dnsPolicy` legt ein Pod fest, welche DNS-Konfiguration er verwendet. Kubernetes unterstützt vier Werte:

* `ClusterFirst` – Standardwert. DNS-Anfragen, die nicht auf die Cluster-Domain passen (z. B. `www.kubernetes.io`), werden an den vom Node geerbten Upstream-Nameserver weitergereicht.
* `Default` – der Pod übernimmt die Namensauflösung direkt vom Node, auf dem er läuft, statt CoreDNS zu nutzen.
* `ClusterFirstWithHostNet` – für Pods mit `hostNetwork: true`, die trotzdem Cluster-DNS nutzen sollen; ohne diese explizite Policy würden sie sonst auf die Node-Auflösung zurückfallen.
* `None` – ignoriert jede DNS-Konfiguration aus dem Cluster-Umfeld, alle Einstellungen müssen dann über `dnsConfig` kommen.

Reicht `dnsPolicy` nicht aus, lässt sich mit `spec.dnsConfig` gezielt nachjustieren, etwa um einen zusätzlichen Nameserver oder eine eigene Search-Domain zu ergänzen:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-config-demo
spec:
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 1.1.1.1
    searches:
      - my-namespace.svc.cluster.local
      - svc.cluster.local
      - cluster.local
    options:
      - name: ndots
        value: "2"
  containers:
  - name: app
    image: my-app:1.0
```

`nameservers` erlaubt maximal drei IP-Adressen und wird mit den aus der `dnsPolicy` erzeugten Basis-Nameservern zusammengeführt (bei `None` ist mindestens ein Eintrag Pflicht). `searches` ergänzt die Basis-Search-Domains, maximal 32 Einträge. `options` ist eine Liste von Objekten mit Pflichtfeld `name` und optionalem `value`, die mit den aus der Policy erzeugten Optionen zusammengeführt werden.

## Discovery live prüfen

Um Discovery an einem laufenden Cluster nachzuvollziehen, lassen sich Service und die dahinterliegenden Adressen direkt gegenüberstellen:

```shell
kubectl get endpoints backend-service
kubectl get endpointslices -l kubernetes.io/service-name=backend-service
```

Für die DNS-Auflösung selbst hilft ein kurzlebiger Debug-Pod mit Netzwerk-Tools:

```shell
kubectl run dns-debug --rm -it --image=busybox:1.36 --restart=Never -- sh
```

Im Container dann gegen den Service-DNS-Namen auflösen:

```shell
nslookup backend-service.my-namespace.svc.cluster.local
dig backend-service.my-namespace.svc.cluster.local
```

Ändert sich die Menge oder Readiness der dahinterliegenden Pods, ändert sich auch das Ergebnis dieser Abfragen, ohne dass an Service oder Pod-Konfiguration etwas angepasst werden muss.

## Abgrenzung

Das DNS-Grundschema (`<service-name>.<namespace>.svc.cluster.local`), Environment-Variable-Discovery und Headless Services behandelt [k8s-development Kapitel 06 Service](../k8s-development/6-k8s-development-service.md). CoreDNS als Cluster-Komponente, kube-proxy und das CNI-Grundmodell behandelt [k8s-admin Kapitel 03 Network](../k8s-admin/3-k8s-admin-network.md). Die Service-Typen ClusterIP, NodePort, LoadBalancer und ExternalName behandelt [k8s-basic Kapitel 09 Service](../k8s-basic/9-k8s-basic-service.md).
