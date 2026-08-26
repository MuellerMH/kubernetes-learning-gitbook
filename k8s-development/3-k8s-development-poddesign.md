# Kubernetes development: Pod design

|||
|---|---|
| Title | K8 Development Pod Design |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-development-training-poddesign](https://muellermh.wordpress.com/k8s-development-training-poddesign) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) |
| Description | Diese Lektion vermittelt Node-Placement per Label, die Steuerung von Rolling Updates sowie Jobs und CronJobs für einmalige und wiederkehrende Workloads. |

## Label, Selector und Annotation

Was Labels, Selectors und Annotations sind und wie sie syntaktisch aufgebaut werden, behandeln bereits [k8s-basic Kapitel 11 Label](../k8s-basic/11-k8s-basic-label.md) und [k8s-basic Kapitel 10 Annotations](../k8s-basic/10-k8s-basic-annotations.md). Aus Entwicklersicht sind Labels vor allem das Werkzeug, mit dem `Deployment`, `Service` und `NetworkPolicy` überhaupt erst wissen, welche Pods zu ihnen gehören — ein falsch gesetztes oder vergessenes Label ist einer der häufigsten Gründe, warum ein neu ausgerollter Pod von seinem Service nicht erreicht wird. In der Praxis lohnt sich ein festes, projektweit einheitliches Set an Labels (z. B. `app`, `tier`, `release`), das über alle Objekte hinweg konsistent gepflegt wird, statt es pro Deployment neu zu erfinden.

## Node selector

Mit `nodeSelector` lässt sich im Pod-Spec erzwingen, dass ein Pod nur auf Nodes mit passendem Label geplant wird:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  nodeSelector:
    hardware-type: gpu
  containers:
  - name: app
    image: my-gpu-app:1.0
```

Ein typischer Anwendungsfall ist das Erzwingen spezieller Hardware, etwa ein Workload, der zwingend auf einem Node mit GPU laufen muss. Der Scheduler platziert den Pod ausschließlich auf Nodes, die das Label `hardware-type=gpu` tragen; existiert kein solcher Node, bleibt der Pod im Status `Pending`.

`nodeSelector` unterstützt nur exakte Gleichheit auf einem Label. Für komplexere Anforderungen wie "bevorzugt, aber nicht zwingend" oder Kombinationen mehrerer Bedingungen bietet Kubernetes zusätzlich `nodeAffinity`, das feiner steuerbare Regeln (`requiredDuringSchedulingIgnoredDuringExecution` / `preferredDuringSchedulingIgnoredDuringExecution`) erlaubt.

## Deployments, Rolling Updates und Rollbacks

Was ein Deployment ist und wie Rollout, Rollback und Skalierung per `kubectl` gesteuert werden, behandelt bereits [k8s-basic Kapitel 08 Deployment](../k8s-basic/8-k8s-basic-deployment.md). Aus Entwicklersicht ist vor allem entscheidend, **wie** ein Rollout abläuft, nicht nur, wie er angestoßen wird. Das steuert `spec.strategy` im Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-demo
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: rolling-demo
  template:
    metadata:
      labels:
        app: rolling-demo
    spec:
      containers:
      - name: app
        image: my-app:2.0
```

`strategy.type` kennt zwei Werte:

* `RollingUpdate` (Default) — alte Pods werden schrittweise durch neue ersetzt, es gibt zu keinem Zeitpunkt einen kompletten Ausfall.
* `Recreate` — alle alten Pods werden zuerst beendet, danach erst die neuen gestartet. Sinnvoll, wenn alte und neue Version nicht parallel laufen dürfen, z. B. bei inkompatiblen Datenbank-Migrationen.

Bei `RollingUpdate` steuern zwei Werte das Tempo und die Sicherheitsmarge des Rollouts:

* `maxSurge` — wie viele Pods zusätzlich zur gewünschten Replica-Zahl während des Rollouts kurzzeitig laufen dürfen (absolut oder als Prozentwert)
* `maxUnavailable` — wie viele Pods während des Rollouts maximal gleichzeitig nicht verfügbar sein dürfen

Ein niedriger `maxUnavailable`-Wert macht den Rollout vorsichtiger und langsamer, ein höherer `maxSurge`-Wert beschleunigt ihn auf Kosten von kurzzeitig mehr laufenden Pods.

## Jobs und CronJobs

Deployments halten dauerhaft laufende Pods am Leben. Für Workloads, die einmalig bis zum Erfolg durchlaufen und sich dann beenden sollen, ist ein `Job` das passende Objekt.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: migration-job
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 4
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: migrate
        image: my-app-migrate:1.0
```

Drei Felder steuern das Verhalten eines Jobs:

* `completions` — wie viele erfolgreiche Pod-Durchläufe insgesamt erreicht werden müssen, bevor der Job als abgeschlossen gilt
* `parallelism` — wie viele Pods dabei gleichzeitig laufen dürfen
* `backoffLimit` — wie oft ein fehlgeschlagener Pod neu gestartet wird, bevor der Job als fehlgeschlagen markiert wird

Ein Job mit `restartPolicy: OnFailure` startet den Container bei einem Fehler erneut im selben Pod; alternativ erzeugt `restartPolicy: Never` bei jedem Fehlversuch einen neuen Pod.

Soll ein Job nicht einmalig, sondern nach einem Zeitplan wiederholt ausgeführt werden, kapselt ein `CronJob` genau das:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-cleanup
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: cleanup
            image: my-cleanup-job:1.0
```

`schedule` folgt dem klassischen Cron-Format. `concurrencyPolicy` legt fest, was passiert, wenn ein neuer Lauf ansteht, während der vorherige noch nicht fertig ist:

Wert | Beschreibung
--- | ---
Allow | mehrere Läufe dürfen parallel existieren (Default)
Forbid | ein neuer Lauf wird übersprungen, solange der vorherige noch aktiv ist
Replace | der laufende Job wird abgebrochen und durch den neuen ersetzt
