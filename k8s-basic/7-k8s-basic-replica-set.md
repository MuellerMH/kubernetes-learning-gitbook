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

Das ReplicaSet ist die weiterentwicklung der Replica Controller. Der haupt Unterschied zwischen dem Replica Controller und dem ReplicaSet ist die unterstützung von Filtern ( selector support ). Das ReplicaSet unterstützt Labels als Selector während der Replica Controller nur gleichheitsbassierte Selektion unterstützt.

## Horizontal Pod Autoscaller

Zudem kann ein ReplicaSet auch in ein HorizontalPodAutoscaler definiert werden. Hier werden dann min und max Replicas definiert und zusätlich ein Wert der das Autoscalling beeinflusst.

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

## Alternativen zum ReplicaSet

- [Deployment](8-k8s-basic-deployment.md)
- [Bare Pods](14-k8s-basic-bare-pod.md)
- [Job](12-k8s-basic-job.md)
- [DeamonSet](13-k8s-basic-daemonset.md)