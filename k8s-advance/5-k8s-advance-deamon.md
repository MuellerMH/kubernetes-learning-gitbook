# Kubernetes advance: DaemonSet

|||
|---|---|
| Title | K8 Advance DaemonSet |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-advance-training-daemonset](https://muellermh.wordpress.com/k8s-advance-training-daemonset) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema DaemonSet |

## DaemonSet

Ein DaemonSet sorgt dafür, dass auf jedem (oder einer definierten Auswahl von) Node im Cluster genau eine Kopie eines bestimmten Pods läuft. Kommt ein neuer Node zum Cluster hinzu, wird automatisch auch dort ein Pod gestartet. Wird ein Node aus dem Cluster entfernt, wird der zugehörige Pod mit entfernt.

Typische Anwendungsfälle für ein DaemonSet sind Aufgaben, die auf jedem Node einzeln laufen müssen:

* Log-Collector (z. B. fluentd), der Logs von jedem Node einsammelt
* Monitoring Agent, der Node-Metriken erfasst
* Storage- oder Netzwerk-Daemon, der auf jedem Node lokale Infrastruktur bereitstellt

## DaemonSet vs. Deployment/ReplicaSet

Ein [Deployment](../k8s-basic/8-k8s-basic-deployment.md) beziehungsweise das darunterliegende [ReplicaSet](../k8s-basic/7-k8s-basic-replica-set.md) sorgt für eine feste, konfigurierte Anzahl an Pod-Replicas, die vom Scheduler auf beliebige Nodes verteilt werden, je nach verfügbaren Ressourcen. Wie viele Replicas auf welchem Node landen, ist dabei nicht garantiert und kann sich auch mehrfach auf demselben Node befinden.

Ein DaemonSet kennt dagegen keine Replica-Anzahl. Die Anzahl der Pods ergibt sich automatisch aus der Anzahl passender Nodes im Cluster: pro Node genau ein Pod, nicht mehr und nicht weniger. Ein DaemonSet ist damit die richtige Wahl, wenn eine Aufgabe node-gebunden ist, ein Deployment die richtige Wahl, wenn nur eine bestimmte Gesamtzahl an Pods irgendwo im Cluster laufen soll.

## DaemonSet beschreiben

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-monitor
  namespace: kube-system
  labels:
    app: node-monitor
spec:
  selector:
    matchLabels:
      app: node-monitor
  template:
    metadata:
      labels:
        app: node-monitor
    spec:
      containers:
      - name: node-monitor
        image: node-monitor:v1
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

Wie beim Deployment wählt `selector.matchLabels` die vom DaemonSet verwalteten Pods anhand ihrer Labels aus, das `template` beschreibt die Pod Vorlage, die auf jedem passenden Node gestartet wird.

## Nodes gezielt auswählen

Soll ein DaemonSet nicht auf jedem, sondern nur auf bestimmten Nodes laufen, lässt sich das über `nodeSelector` in der Pod Vorlage einschränken:

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
```

In diesem Fall läuft der DaemonSet-Pod nur auf Nodes, die das Label `disktype: ssd` tragen.
