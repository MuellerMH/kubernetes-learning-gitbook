# Kubernetes advance: Horizontal Pod Autoscaling

|||
|---|---|
| Title | K8 Advance Horizontal Pod Autoscaling |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-advance-training-autoscaling](https://muellermh.wordpress.com/k8s-advance-training-autoscaling) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Horizontal Pod Autoscaling |

## Horizontal Pod Autoscaling

Der HorizontalPodAutoscaler (HPA) passt automatisch die Anzahl der Replicas eines Deployments, ReplicaSets oder StatefulSets an, basierend auf beobachteten Metriken wie CPU- oder Memory-Auslastung. Steigt die Last, werden mehr Pods gestartet, sinkt sie wieder, werden überzählige Pods entfernt. Das ist horizontale Skalierung, im Unterschied zur vertikalen Skalierung, bei der einem bestehenden Pod mehr Ressourcen zugewiesen werden.

Der HPA arbeitet als Regelkreis, der in einem festen Intervall (per Default alle 15 Sekunden) die aktuellen Metriken abfragt und daraus die gewünschte Anzahl an Replicas berechnet. Weicht der aktuelle Wert deutlich genug vom Zielwert ab, wird skaliert.

## Voraussetzung: Metrics Server

Damit der HPA überhaupt CPU- und Memory-Auslastung von Pods abfragen kann, muss im Cluster ein Metrics Server laufen. Der Metrics Server ist kein Bestandteil einer Standard-Kubernetes-Installation und muss separat installiert werden. Ohne laufenden Metrics Server kann der HPA keine Resource-Metriken beziehen und nicht skalieren.

## HPA erstellen

### Imperativ mit kubectl

```shell
kubectl autoscale deployment web-app --cpu-percent=50 --min=2 --max=10
```

Dieser Befehl erstellt einen HPA für das Deployment `web-app`, der die Anzahl der Replicas zwischen 2 und 10 hält und versucht, die durchschnittliche CPU-Auslastung der Pods bei 50 % der angeforderten CPU (`requests.cpu`) zu halten.

### Deklarativ mit YAML

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

`scaleTargetRef` verweist auf das zu skalierende Deployment. `minReplicas` und `maxReplicas` begrenzen den Skalierungsbereich nach unten und oben, damit weder alle Pods abgebaut werden noch der Cluster unbegrenzt wächst. Damit die Metrik `cpu` ausgewertet werden kann, müssen für die Container des Ziel-Deployments zudem [Resource Requests](6-k8s-advance-resources.md) für CPU gesetzt sein, da sich `averageUtilization` prozentual auf `requests.cpu` bezieht.
