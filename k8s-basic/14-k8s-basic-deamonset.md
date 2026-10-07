# Kubernetes basics: DaemonSets

|||
|---|---|
| Title | K8 Basic DaemonSets |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) |
| Description | In dieser Lektion lernst du was ein DaemonSet ist, wofür es eingesetzt wird und wie es sich vom ReplicaSet unterscheidet. |

## DaemonSets

Ein DaemonSet (`apps/v1`) sorgt dafür, dass auf **jedem** Node des Clusters (oder auf einer ausgewählten Teilmenge der Nodes) genau eine Kopie eines Pods läuft. Kommt ein neuer Node in den Cluster, startet der DaemonSet Controller dort automatisch einen Pod. Wird ein Node entfernt, wird der zugehörige Pod mit aufgeräumt.

Anders als beim [ReplicaSet](7-k8s-basic-replica-set.md) gibt es deshalb kein `replicas` Feld: Die Anzahl der Pods ergibt sich allein aus der Anzahl der passenden Nodes.

Typische Einsatzzwecke sind Hilfsprozesse, die auf Node-Ebene arbeiten:

- Log-Collector wie Fluent Bit oder Fluentd, die die Logs aller Container eines Nodes einsammeln
- Monitoring-Agenten wie der Prometheus Node Exporter
- Netzwerk-Plugins (CNI) wie Calico oder Cilium und der `kube-proxy`
- Storage-Daemons, zum Beispiel für Ceph oder CSI Treiber

## DaemonSet beschreiben

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter
        ports:
        - containerPort: 9100
```

Wie beim ReplicaSet und Deployment müssen die Labels im `selector` zu den Labels im Pod Template passen.

## Arbeiten mit DaemonSets

```shell
# DaemonSet erstellen
kubectl apply -f node-exporter.yaml

# DaemonSets anzeigen (DESIRED entspricht der Anzahl passender Nodes)
kubectl get daemonsets

# Pods inklusive Node-Zuordnung anzeigen
kubectl get pods -l app=node-exporter -o wide

# DaemonSet löschen (entfernt auch alle zugehörigen Pods)
kubectl delete daemonset node-exporter
```

Eine Abkürzung für `daemonsets` ist `ds`, also zum Beispiel `kubectl get ds`.

Wie ein DaemonSet nur auf bestimmten Nodes läuft (`nodeSelector`, Tolerations für Master/Control-Plane Nodes) und wie Updates ausgerollt werden, erfährst du im Kapitel [Advance: DaemonSet](../k8s-advance/5-k8s-advance-deamon.md).

## Alternativen zum DaemonSet

- [ReplicaSet](7-k8s-basic-replica-set.md)
- [Deployment](8-k8s-basic-deployment.md)
- [Job](13-k8s-basic-job.md)
- [Bare Pods](15-k8s-basic-bare-pod.md)
