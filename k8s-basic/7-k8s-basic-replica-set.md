# Kubernetes basics: Replica Set

|||
|---|---|
| Title | K8 Basic Replica Sets |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Replica Sets |

## ReplicaSet

Das ReplicaSet ist die Weiterentwicklung des Replication Controllers. Der Hauptunterschied zwischen dem Replica Controller und dem ReplicaSet ist die Unterstützung von mengenbasierten Selektoren (set-based selector). Das ReplicaSet unterstützt neben gleichheitsbasierter Selektion auch Ausdrücke wie `In` oder `NotIn`, während der Replication Controller nur gleichheitsbasierte Selektion unterstützt.

In der Praxis wird ein ReplicaSet selten direkt angelegt. Meistens wird ein [Deployment](8-k8s-basic-deployment.md) erstellt, das seinerseits ReplicaSets erzeugt und verwaltet, zum Beispiel um Rolling Updates und Rollbacks zu ermöglichen.

## ReplicaSet beschreiben

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: environment, operator: In, values: [dev, prod]}
  template:
    metadata:
      labels:
        tier: frontend
        environment: dev
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Neben `matchLabels` unterstützt der Selector mit `matchExpressions` auch mengenbasierte Abfragen mit den Operatoren `In`, `NotIn`, `Exists` und `DoesNotExist`. Die Labels im Pod Template müssen zum Selector passen, sonst lehnt die API das ReplicaSet ab.

## Arbeiten mit ReplicaSets

```shell
# ReplicaSet erstellen
kubectl apply -f frontend.yaml

# ReplicaSets anzeigen (Kurzform: rs)
kubectl get rs

# auf 5 Replicas skalieren
kubectl scale rs frontend --replicas=5

# ReplicaSet löschen, die Pods aber weiterlaufen lassen
kubectl delete rs frontend --cascade=orphan
```

Ein ReplicaSet übernimmt auch bereits laufende Pods, deren Labels zum Selector passen, selbst wenn diese nicht von ihm erzeugt wurden. Ein [Bare Pod](15-k8s-basic-bare-pod.md) mit dem Label `tier: frontend` würde also mitgezählt.

## Horizontal Pod Autoscaler

Zudem kann ein ReplicaSet als Ziel eines HorizontalPodAutoscalers definiert werden. Hier werden min und max Replicas festgelegt und zusätzlich ein Zielwert, der das Autoscaling steuert. Mehr dazu im Kapitel [Advance: Autoscaling](../k8s-advance/7-k8s-advance-autoscalling.md).

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

## Alternativen zum ReplicaSet

- [Deployment](8-k8s-basic-deployment.md)
- [Bare Pods](15-k8s-basic-bare-pod.md)
- [Job](13-k8s-basic-job.md)
- [DeamonSet](14-k8s-basic-deamonset.md)