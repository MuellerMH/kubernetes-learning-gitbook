# Kubernetes basics:  etcd

|||
|---|---|
| Title | K8 Basic Etcd |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description | Am Ende dieser Lektion wirst du die grundlegenden Themen der k8s etcd verstehen. Was ist die Kubernetes etcd ? |

---

## etcd

Der etcd ist die hochverfügbare und konsistente Key Value Datenspeicher und somit das Gehirn des Kubernetes Clusters. Für ein kurzen Testbetrieb, bei dem der permanente Status des Clusters nicht relevant ist reicht ein einziger etcd prozess im Cluster. Dies ist jedoch nicht empfehlenswert wenn ein Cluster permanent oder soagar produktiv laufen soll. Eigentlich kann der etcd Key-Value Storage auf allen Nodes und Mastern laufen, jedoch hat sich auch hier in der Praxis bewährt diese auf seperaten Instanzen laufen zu lassen um eine optimale verfügbarkeit zu garantieren. Das kleinste ausfallsichere Setup sind 3 etcd Instanzen. Die Failure tolerance entspricht folgender auflistung.

| Instanzen | Quorum | Failure Tolerance |
|---|---|---|
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |
| 6 | 4 | 2 |
| 7 | 4 | 3 |

Das Quorum ist die Mehrheit der Instanzen (`n/2 + 1`), die erreichbar sein muss, damit der Cluster Schreibzugriffe annimmt.

Dadurch ergibt sich eine optimale Anzahl von ungeraden Instanzen (1, 3, 5, 7), da eine gerade Anzahl an Instanzen keinen Mehrwert bietet. In der Praxis werden meist 3 oder 5 Instanzen betrieben, da mit jeder weiteren Instanz auch der Aufwand für die Replikation der Schreibzugriffe steigt.

## Backup

Das Backup des etcd Speichers ist eines der wichtigsten Task, durch dieses Backup kann ein Cluster jederzeit wieder zu dem gespeicherten Status gebracht werden, da alle Komponenten des Clusters, bis auf die Volumes, stateless sind und somit nur das bereitstellen, was dem Kubernetes Cluster im etcd Speicher bekannt ist.

Das Backup erstellt natürlich auch die Volumes wieder her, aber nicht den Inhalt, die darauf vorher verfügbar war. Die Volumes müssen seperat gebackupt werden. Darunter fallen alle Arten von Permanenten Speicher arten.

### Snapshot erstellen

Das Backup wird mit `etcdctl` als Snapshot erstellt. Da der etcd per TLS abgesichert ist, müssen die Zertifikate mit übergeben werden. In einem mit `kubeadm` aufgesetzten Cluster liegen diese unter `/etc/kubernetes/pki/etcd/`:

```shell
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Die benötigten Pfade und Endpunkte findest du im Manifest des etcd Static Pods unter `/etc/kubernetes/manifests/etcd.yaml`.

Ob der Snapshot gültig ist, lässt sich anschließend prüfen:

```shell
etcdutl snapshot status /backup/etcd-snapshot.db --write-out=table
```

### Snapshot wiederherstellen

Beim Restore wird aus dem Snapshot ein neues Datenverzeichnis erzeugt:

```shell
etcdutl snapshot restore /backup/etcd-snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

Danach wird im etcd Manifest `/etc/kubernetes/manifests/etcd.yaml` der `hostPath` des Volumes `etcd-data` auf das neue Verzeichnis `/var/lib/etcd-from-backup` umgestellt. Das Kubelet erkennt die Änderung und startet den etcd Pod mit den wiederhergestellten Daten neu.

> Tipp: Backup und Restore des etcd sind ein klassischer Bestandteil der CKA-Prüfung. Übe beide Schritte auf einem Test-Cluster, bis du sie ohne Nachschlagen beherrschst.

Weitere Informationen zum hochverfügbaren Betrieb des etcd findest du im Kapitel [Admin: High Availability](../k8s-admin/4-k8s-admin-highavailabity.md).
