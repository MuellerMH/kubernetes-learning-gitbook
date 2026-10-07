# Kubernetes admin: Cluster Maintenance

|||
|---|---|
| Title | K8 Admin Cluster Maintenance |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-admin-training-maintenance](https://muellermh.wordpress.com/k8s-admin-training-maintenance) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) |
| Description | Diese Lektion vermittelt die wichtigsten Wartungsaufgaben eines Clusters: Upgrades mit kubeadm, etcd Backups, Zertifikate und Troubleshooting. |

## Versionierung

Kubernetes veröffentlicht etwa alle vier Monate eine neue Minor Version (z. B. `1.30` → `1.31`). Unterstützt werden jeweils die drei aktuellsten Minor Versionen. Ein Cluster sollte deshalb regelmäßig aktualisiert werden.

Dabei gelten zwei wichtige Regeln:

* Upgrades erfolgen immer nur **eine Minor Version** auf einmal. Von `1.29` auf `1.31` geht es also über `1.30`.
* Die Control Plane wird **zuerst** aktualisiert, danach die Worker Nodes. Das Kubelet darf älter als der API Server sein (bis zu drei Minor Versionen), aber niemals neuer.

## Upgrade mit kubeadm

### Control Plane

Auf dem ersten Control Plane Node wird zuerst `kubeadm` selbst aktualisiert. Die Paketquellen sind pro Minor Version getrennt, deshalb muss vorher das Repository auf die neue Version umgestellt werden (z. B. in `/etc/apt/sources.list.d/kubernetes.list`).

```shell
# kubeadm aktualisieren
apt-get update
apt-get install -y kubeadm='1.31.x-*'

# Upgrade planen: zeigt die möglichen Zielversionen und prüft den Cluster
kubeadm upgrade plan

# Upgrade der Control Plane Komponenten durchführen
kubeadm upgrade apply v1.31.x
```

Auf weiteren Control Plane Nodes wird statt `upgrade apply` der Befehl `kubeadm upgrade node` ausgeführt.

Danach werden Kubelet und kubectl des Nodes aktualisiert. Damit währenddessen keine Workloads gestört werden, wird der Node vorher geleert:

```shell
kubectl drain controlplane --ignore-daemonsets

apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*'
systemctl daemon-reload
systemctl restart kubelet

kubectl uncordon controlplane
```

### Worker Nodes

Die Worker Nodes werden nacheinander aktualisiert, damit immer genug Kapazität für die Workloads vorhanden ist:

```shell
# vom Control Plane Node aus
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data

# auf node-1
apt-get install -y kubeadm='1.31.x-*'
kubeadm upgrade node
apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*'
systemctl daemon-reload
systemctl restart kubelet

# wieder vom Control Plane Node aus
kubectl uncordon node-1
```

Mit `kubectl get nodes` siehst du in der Spalte `VERSION`, welcher Node bereits aktualisiert wurde. Details zu `drain`, `cordon` und `uncordon` findest du im Kapitel [Nodes](5-k8s-admin-nodes.md).

## etcd Backup

Vor jedem Upgrade, und natürlich regelmäßig, sollte ein Snapshot des etcd erstellt werden. Wie Snapshots mit `etcdctl` erstellt und mit `etcdutl` wiederhergestellt werden, ist im Kapitel [Basic: etcd](../k8s-basic/2-k8s-basic-etcd.md#backup) beschrieben.

Zusätzlich lohnt es sich, die Definitionen aller Ressourcen zu sichern, zum Beispiel über ein Git Repository (GitOps) oder mit Tools wie [Velero](https://velero.io/), das auch Volumes sichern kann:

```shell
kubectl get all --all-namespaces -o yaml > all-resources.yaml
```

## Zertifikate

Die Komponenten der Control Plane kommunizieren per TLS. Die von `kubeadm` erzeugten Zertifikate liegen unter `/etc/kubernetes/pki/` und sind standardmäßig **ein Jahr** gültig (die CA zehn Jahre). Bei jedem `kubeadm upgrade` werden sie automatisch erneuert. Wird ein Cluster länger nicht aktualisiert, müssen sie manuell erneuert werden:

```shell
# Ablaufdaten anzeigen
kubeadm certs check-expiration

# alle Zertifikate erneuern
kubeadm certs renew all
```

Nach dem Erneuern müssen die Static Pods der Control Plane (API Server, Controller Manager, Scheduler, etcd) neu gestartet werden, damit sie die neuen Zertifikate laden. Am einfachsten verschiebst du dazu ihre Manifeste kurz aus `/etc/kubernetes/manifests/` heraus und wieder hinein.

Ein einzelnes Zertifikat lässt sich mit `openssl` untersuchen:

```shell
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

## Troubleshooting

### Control Plane

In einem kubeadm Cluster laufen die Control Plane Komponenten als Static Pods im Namespace `kube-system`. Ihre Manifeste liegen unter `/etc/kubernetes/manifests/`, das Kubelet startet sie direkt ohne Umweg über den API Server.

```shell
kubectl get pods -n kube-system
kubectl logs -n kube-system kube-apiserver-controlplane
```

Ist der API Server selbst nicht erreichbar, hilft `kubectl` nicht weiter. Dann geht es direkt über die Container Runtime:

```shell
crictl ps -a
crictl logs <container-id>
```

### Worker Nodes

Steht ein Node auf `NotReady`, liegt die Ursache meist beim Kubelet, bei der Container Runtime oder beim Netzwerk-Plugin:

```shell
kubectl describe node node-1    # Conditions und Events prüfen

# auf dem Node selbst
systemctl status kubelet
journalctl -u kubelet -f
systemctl status containerd
```

Häufige Ursachen sind ein gestoppter Kubelet Dienst, abgelaufene Zertifikate, eine fehlerhafte Konfiguration in `/var/lib/kubelet/config.yaml` oder ein volles Dateisystem.

### Workloads

Für Probleme mit Pods und Services gilt die Reihenfolge `get` → `describe` → `logs` → `exec`:

```shell
kubectl get pods -o wide
kubectl describe pod <pod>          # Events: Scheduling, Image Pull, Probes
kubectl logs <pod> --previous       # Logs des letzten abgestürzten Containers
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get endpoints <service>     # hat der Service überhaupt Pods?
```

Weitere Debugging-Techniken aus Entwicklersicht findest du im Kapitel [Development: Observability](../k8s-development/5-k8s-development-observability.md).
