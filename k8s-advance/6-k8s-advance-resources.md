# Kubernetes advance: Resources

|||
|---|---|
| Title | K8 Advance Resources |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-advance-training-resources](https://muellermh.wordpress.com/k8s-advance-training-resources) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Resource Requests und Limits |

## Resources

Für jeden Container in einem Pod lassen sich CPU- und Memory-Bedarf über `requests` und `limits` angeben. Beide steuern unterschiedliche Dinge und werden an unterschiedlichen Stellen im Cluster ausgewertet.

* `requests` ist der Bedarf, den der Scheduler garantiert einplant. Anhand der `requests` entscheidet der Scheduler, auf welchem Node genug freie Kapazität vorhanden ist, um den Pod zu platzieren.
* `limits` ist die Obergrenze, die der Kubelet auf dem Node durchsetzt. Ein Container darf zur Laufzeit nicht mehr Ressourcen verbrauchen, als in `limits` definiert ist.

## Resources beschreiben

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: 250m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 256Mi
```

CPU wird in Cores angegeben, wobei `1` einem vollen Core entspricht. `250m` steht für 250 Millicores, also ein Viertel Core. Memory wird in Bytes angegeben, üblicherweise mit Suffixen wie `Mi` (Mebibyte) oder `Gi` (Gibibyte).

## Auswirkung bei Überschreitung

CPU und Memory verhalten sich bei Überschreitung des `limits` Werts unterschiedlich:

* **CPU:** CPU ist eine komprimierbare Ressource. Versucht ein Container mehr CPU zu nutzen, als in `limits.cpu` erlaubt ist, wird er nicht beendet, sondern gedrosselt (Throttling). Der Container läuft weiter, bekommt aber keine zusätzliche CPU-Zeit über das Limit hinaus zugeteilt.
* **Memory:** Memory ist nicht komprimierbar. Überschreitet ein Container den in `limits.memory` gesetzten Wert, wird der Container vom Kernel mit einem Out-Of-Memory Kill (OOMKill) beendet. Kubernetes startet den Container danach entsprechend der Restart Policy neu.

## Verhältnis requests zu limits

Nur wenn für jeden Container im Pod sowohl bei CPU als auch bei Memory `requests` und `limits` gesetzt und jeweils identisch sind, erhält der Pod die Quality of Service Klasse `Guaranteed` und läuft damit am stabilsten unter Ressourcendruck auf dem Node. Ist `requests` niedriger als `limits`, kann der Container kurzfristig mehr Ressourcen nutzen als angefordert, solange auf dem Node Kapazität frei ist, riskiert bei Memory-Überschreitung aber weiterhin ein OOMKill.
