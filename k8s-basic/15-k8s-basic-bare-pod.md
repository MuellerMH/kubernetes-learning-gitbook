# Kubernetes basics: Bare Pods

|||
|---|---|
| Title | K8 Basic Bare Pods |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description | In dieser Lektion lernst du was ein Bare Pod ist, warum er in den vorherigen Kapiteln als Alternative gelistet wird und wann sein Einsatz trotzdem sinnvoll ist. |

## Bare Pods

Ein Bare Pod ist ein Pod, der direkt und ohne Controller (ReplicaSet, Deployment, Job, ...) erzeugt wurde. Es gibt also keine Instanz, die sich um ihn kümmert.

Genau deshalb taucht er in den Kapiteln zum ReplicationController und ReplicaSet als Negativbeispiel bzw. Alternative auf: Fällt die Node aus oder crasht der Container, wird der Pod nicht neu geplant. Es gibt kein Self-Healing, keine Skalierung und kein Rolling Update, da schlicht kein Controller existiert, der diese Aufgaben übernehmen könnte.

Ein nackter Pod ohne Controller sieht so aus:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  containers:
  - name: debug
    image: busybox
    command: ["sleep", "3600"]
```

Für Produktions-Workloads ist das in aller Regel keine gute Wahl, da ein Ausfall des Pods manuell bemerkt und behoben werden muss. Legitim ist ein Bare Pod dagegen für kurzlebige Debug- oder Test-Pods, die du gezielt einmalig hochfährst und danach wieder wegwirfst.