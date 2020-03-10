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

1 & 2 Instance = 0 Failure Tolerance
3 & 4 Instance = 1 Failure Tolerance
5 & 6 Instance = 2 Failure Tolerance
7 & 8 Instance = 3 Failure Tolerance
9     Instance = 4 Failure Tolerance

Dadurch ergebit sich eine optimale Anzahl von ungeraden Instanzen 1,3,5,7,9 da die gerade Anzahl an Instanzen keinen mehrwert bietet.

## Backup

Das Backup des etcd Speichers ist eines der wichtigsten Task, durch dieses Backup kann ein Cluster jederzeit wieder zu dem gespeicherten Status gebracht werden, da alle Komponenten des Clusters, bis auf die Volumes, stateless sind und somit nur das bereitstellen, was dem Kubernetes Cluster im etcd Speicher bekannt ist.

Das Backup erstellt natürlich auch die Volumes wieder her, aber nicht den Inhalt, die darauf vorher verfügbar war. Die Volumes müssen seperat gebackupt werden. Darunter fallen alle Arten von Permanenten Speicher arten.