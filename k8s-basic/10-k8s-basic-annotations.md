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
| Description | In dieser Lektion lernst du was Annotationen sind und wpfür sie da sind und wie diese eingesetzt werden können. |

## Annotations

Solltest du das CKA Zertifikat anstreben ist der Annotation teil nicht gerade unbedeutend und erst recht im Cluster selbst eine starke Bedeutung hat. Auch wenn die Dokumentation sehr klein ausfällt.

Im Gegensatz zu Labels die zum selektieren genutzt werden, bieteten Annotations eine freizügigere Bezeichnug. Hier können belibige Key Value Pairs in strukturierter und unstrucktierter weise definert werden.

Ein Beispiel hierfür wäre zum Beispiel das setzen der letzten Release ID, der Git Branch oder eine verantwortliche Person oder informationen zu prometheus.

```yaml
metadata:
  annotations:
    prometheus.io/path: /metrics
    prometheus.io/port: "9102"
    prometheus.io/scheme: http
    prometheus.io/scrape: "false"
```

Diese Annotations können dann von anderen Applikationen über die API abgefragt werden.