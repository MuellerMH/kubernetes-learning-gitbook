# Kubernetes basics: Jobs

|||
|---|---|
| Title | K8 Basic Jobs |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description | In dieser Lektion lernst du was ein Kubernetes Job ist, wodurch er sich von ReplicaSet und Deployment unterscheidet und wie eine minimale Job Definition aussieht. |

## Jobs

Ein Job (`batch/v1`) startet einen oder mehrere Pods und kümmert sich darum, dass diese bis zum erfolgreichen Abschluss laufen. Anders als bei einem ReplicaSet oder Deployment ist ein Job nicht auf Dauerbetrieb ausgelegt: Sobald die Pods erfolgreich terminiert sind, gilt der Job als abgeschlossen und es werden keine neuen Pods mehr nachgestartet.

Zur Abgrenzung:

- Ein **Bare Pod** läuft einmalig, es gibt kein Neustart-Management. Stirbt er, bleibt er tot.
- Ein **ReplicaSet** hält dauerhaft eine bestimmte Anzahl Pods am Leben, das Ziel ist Dauerbetrieb.
- Ein **Job** hat als Ziel den erfolgreichen Abschluss der Aufgabe, danach ist Schluss.

Damit eignet sich ein Job vor allem für einmalige oder abgeschlossene Aufgaben, zum Beispiel Batch-Verarbeitung, Migrationen oder Datenexporte.

Eine minimale Job Definition sieht so aus:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

Details zu `completions`, `parallelism`, `backoffLimit` oder `concurrencyPolicy` findest du im Kapitel [k8s-development/3-k8s-development-poddesign.md](../k8s-development/3-k8s-development-poddesign.md).