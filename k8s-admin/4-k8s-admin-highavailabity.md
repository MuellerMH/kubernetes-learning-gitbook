# Kubernetes admin: High Availability

|||
|---|---|
| Title | K8 Admin High Availability |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-admin-training-highavailability](https://muellermh.wordpress.com/k8s-admin-training-highavailability) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema High Availability |

## Warum High Availability

Ein Cluster mit nur einem Control-Plane-Node hat einen Single Point of Failure: Fällt der API-Server, der Scheduler, der Controller-Manager oder etcd auf diesem einen Node aus, ist der Cluster nicht mehr administrierbar. Bereits laufende Workloads auf den Worker-Nodes bleiben zwar zunächst am Leben, aber es lassen sich keine neuen Objekte anlegen, keine Pods neu scheduln und keine Zustandsänderungen mehr durchsetzen, bis die Control-Plane wieder erreichbar ist.

High Availability (HA) verteilt die kritischen Komponenten auf mehrere Nodes, damit der Ausfall eines einzelnen Nodes den Cluster als Ganzes nicht lahmlegt.

## Control-Plane-HA

In einem HA-Setup laufen mehrere API-Server-Instanzen parallel, typischerweise hinter einem Load Balancer, der eingehende Anfragen auf die verfügbaren API-Server verteilt. `kubectl` und alle Cluster-Komponenten sprechen dann mit dem Load Balancer statt mit einem einzelnen API-Server.

Für den Aufbau mit `kubeadm` gibt es zwei etablierte Topologien:

* **Stacked etcd** — etcd läuft auf denselben Nodes wie die übrige Control-Plane. Weniger Nodes nötig, aber der Ausfall eines Nodes betrifft sowohl Control-Plane als auch etcd-Member gleichzeitig.
* **External etcd** — etcd läuft auf eigenen, von der Control-Plane getrennten Nodes. Mehr Nodes nötig, dafür sind Control-Plane- und etcd-Ausfälle voneinander entkoppelt.

```shell
kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:PORT" --upload-certs
```

Weitere Control-Plane-Nodes treten dem bestehenden Cluster anschließend mit einem eigenen Join-Befehl bei:

```shell
kubeadm join LOAD_BALANCER_DNS:PORT --control-plane --certificate-key ...
```

## etcd-HA

etcd ist der Key-Value-Store, in dem der gesamte Cluster-Zustand liegt. Ein etcd-Cluster besteht aus mehreren Membern, die sich per Raft-Konsensprotokoll auf einen gemeinsamen Zustand einigen. Damit Raft bei einem Netzwerk-Split eindeutig eine Mehrheit (Quorum) bestimmen kann, wird etcd immer mit einer **ungeraden Anzahl an Membern** betrieben — üblich sind drei oder fünf. Bei einer geraden Anzahl kann ein Split zu einem Patt führen, bei dem kein Quorum mehr zustande kommt.

Ein etcd-Cluster mit drei Membern verkraftet den Ausfall eines Members, ohne die Schreibfähigkeit zu verlieren; ein Cluster mit fünf Membern verkraftet den Ausfall von zweien.

Für die Datensicherung von etcd existiert das Konzept des Snapshots. `etcdctl` kann einen konsistenten Snapshot des aktuellen Zustands ziehen, der im Notfall zur Wiederherstellung dient:

```shell
etcdctl snapshot save snapshot.db
etcdctl snapshot status snapshot.db
```

Die konkrete Snapshot- und Restore-Prozedur hängt stark vom jeweiligen Cluster-Setup ab und ist hier nur als Konzept genannt, nicht als vollständige Anleitung.

## Worker-Node-Verteilung

HA endet nicht bei der Control-Plane. Auch Worker-Nodes sollten über mehrere Failure-Domains verteilt sein, zum Beispiel über unterschiedliche Verfügbarkeitszonen oder physische Racks, damit der Ausfall einer einzelnen Domain nicht die gesamte Kapazität für Workloads mitreißt. Kubernetes selbst plant Pods nicht automatisch failure-domain-bewusst — dafür sind zusätzliche Scheduling-Mechanismen nötig, die über den Rahmen dieses Kapitels hinausgehen.
