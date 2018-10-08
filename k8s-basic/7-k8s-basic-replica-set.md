# Kubernetes basics: Replica Set

|||
|---|---|
| Title | K8 Basic training |
| Category | Course |
| Level | Novice |
| Duration | ? |
| Url | ... |
| Description | By the end of this course you will understand the basic topics of k8s. You will learn how container, pods, replica, services, secrets, labels and deployments works also with helath-checks.|

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