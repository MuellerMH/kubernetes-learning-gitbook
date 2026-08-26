# Kubernetes admin: Nodes

|||
|---|---|
| Title | K8 Admin Nodes |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-admin-training-nodes](https://muellermh.wordpress.com/k8s-admin-training-nodes) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/architecture/nodes/](https://kubernetes.io/docs/concepts/architecture/nodes/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Node-Administration |

## Self-Registration of Nodes

Ein Node meldet sich im Regelfall selbst am Cluster an. Der `kubelet`-Prozess registriert sich beim Start mit dem Flag `--register-node` (Standard: `true`) automatisch als Node-Objekt am API-Server, mitsamt seiner Kapazität und seinen Labels. Ein Administrator muss den Node also nicht manuell als Objekt anlegen, sobald `kubelet` mit den passenden Zugangsdaten auf dem Node läuft, erscheint er von selbst im Cluster.

```shell
kubectl get nodes
kubectl describe node <node-name>
```

Soll die Selbstregistrierung nicht genutzt werden, etwa weil Nodes vorab zentral verwaltet werden, lässt sich `--register-node=false` setzen; das Node-Objekt muss dann vorher manuell angelegt werden.

## Manual Node Administration

Bevor Wartungsarbeiten an einem Node anstehen (Neustart, Kernel-Update, Hardware-Tausch), sollte der Node aus der Scheduling-Rotation genommen werden:

* `kubectl cordon <node-name>` — markiert den Node als nicht mehr schedulebar. Bereits laufende Pods bleiben unangetastet, es werden nur keine neuen Pods mehr auf diesen Node geplant.
* `kubectl drain <node-name>` — entfernt zusätzlich die laufenden Pods vom Node (unter Berücksichtigung von PodDisruptionBudgets) und verteilt sie auf andere Nodes neu.
* `kubectl uncordon <node-name>` — hebt die Sperre wieder auf, der Node ist wieder regulär schedulebar.

```shell
kubectl cordon node-1
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
kubectl uncordon node-1
```

Technisch setzt `cordon` intern einen Taint (`node.kubernetes.io/unschedulable`) auf den Node. Ein **Taint** auf einem Node stößt Pods ab, die die passende **Toleration** nicht mitbringen — der Scheduler plant keine neuen Pods auf einen getainteten Node, es sei denn, der Pod toleriert diesen Taint explizit:

```yaml
tolerations:
- key: "node.kubernetes.io/unschedulable"
  operator: "Exists"
  effect: "NoSchedule"
```

## Node Capacity

Jeder Node meldet seine Kapazität in zwei Werten:

* `capacity` — die tatsächliche, physisch vorhandene Kapazität des Nodes (CPU, Memory, Pods)
* `allocatable` — die für Pods tatsächlich nutzbare Kapazität, also `capacity` abzüglich der für System- und Kubernetes-eigene Prozesse reservierten Ressourcen

```shell
kubectl describe node <node-name>
```

Der Abschnitt `Capacity` und `Allocatable` in der Ausgabe von `kubectl describe node` zeigt beide Werte nebeneinander.

Für einen schnellen Überblick über die aktuelle Auslastung eines Nodes gibt es zusätzlich:

```shell
kubectl top node
```

> Wichtig: `kubectl top` ist keine eingebaute Funktion des API-Servers. Sie setzt einen separat zu installierenden **Metrics Server** im Cluster voraus. Ohne laufenden Metrics Server liefert `kubectl top node` einen Fehler, unabhängig davon, wie viele Nodes im Cluster registriert sind.

[https://kubernetes.io/docs/concepts/architecture/nodes/](https://kubernetes.io/docs/concepts/architecture/nodes/)
