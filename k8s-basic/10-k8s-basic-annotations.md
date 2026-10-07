# Kubernetes basics: Annotations

|||
|---|---|
| Title | K8 Basic Annotations |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/) |
| Description | In dieser Lektion lernst du was Annotationen sind und wofür sie da sind und wie diese eingesetzt werden können. |

## Annotations

Solltest du das CKA Zertifikat anstreben, ist der Annotation-Teil nicht unbedeutend, und auch im Cluster selbst haben Annotations eine große Bedeutung, auch wenn die Dokumentation dazu sehr klein ausfällt.

Im Gegensatz zu [Labels](11-k8s-basic-label.md), die zum Selektieren genutzt werden, bieten Annotations eine freizügigere Bezeichnung. Hier können beliebige Key-Value-Paare in strukturierter und unstrukturierter Weise definiert werden. Annotations können **nicht** in Selektoren verwendet werden.

Beispiele hierfür sind die letzte Release ID, der Git Branch, eine verantwortliche Person oder Informationen für Prometheus.

```yaml
metadata:
  annotations:
    prometheus.io/path: /metrics
    prometheus.io/port: "9102"
    prometheus.io/scheme: http
    prometheus.io/scrape: "false"
```

Diese Annotations können dann von anderen Applikationen über die API abgefragt werden.

## Annotations mit kubectl setzen

```shell
# Annotation setzen
kubectl annotate pod my-pod owner="team-a"

# bestehende Annotation überschreiben
kubectl annotate pod my-pod owner="team-b" --overwrite

# Annotation entfernen (Minus am Ende des Keys)
kubectl annotate pod my-pod owner-

# Annotations anzeigen
kubectl describe pod my-pod
kubectl get pod my-pod -o jsonpath='{.metadata.annotations}'
```

## Syntax

Der Key einer Annotation besteht wie bei Labels aus einem optionalen Präfix und einem Namen, getrennt durch einen Schrägstrich, zum Beispiel `prometheus.io/scrape`. Das Präfix muss eine DNS-Subdomain sein, der Name darf maximal 63 Zeichen lang sein. Die Präfixe `kubernetes.io/` und `k8s.io/` sind für Kubernetes-eigene Komponenten reserviert. Werte sind immer Strings, deshalb müssen Zahlen und Booleans in Anführungszeichen stehen (`"9102"`, `"false"`).

## Annotations, die Kubernetes selbst nutzt

Viele Komponenten werten Annotations aus oder schreiben sie selbst, einige Beispiele:

| Annotation | Bedeutung |
|---|---|
| `kubernetes.io/change-cause` | wird von `kubectl rollout history` als Grund für eine Revision angezeigt |
| `deployment.kubernetes.io/revision` | aktuelle Revision eines Deployments, wird vom Controller gesetzt |
| `kubectl.kubernetes.io/last-applied-configuration` | letzte mit `kubectl apply` angewendete Konfiguration |
| `nginx.ingress.kubernetes.io/rewrite-target` | Konfiguration des NGINX Ingress Controllers, siehe [Ingress](../k8s-advance/3-k8s-advance-ingress.md) |
| `storageclass.kubernetes.io/is-default-class` | markiert eine StorageClass als Default |

Ein Beispiel für den Rollout-Verlauf eines Deployments:

```shell
kubectl annotate deployment nginx kubernetes.io/change-cause="Update auf nginx 1.27"
kubectl rollout history deployment nginx
```
