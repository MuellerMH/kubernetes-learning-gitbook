# Kubernetes Admin Training Intro

|||
|---|---|
| Title | K8 Admin training |
| Category | Course |
| Level | proficient |
| Duration | ? |
| Url | ... |
| Description | Am Ende dieses Kurses bist du auf die administrativen Themen der CKA-Prüfung vorbereitet. Du lernst, wie man Namespaces, Benutzer und Rechte (RBAC), das Cluster-Netzwerk, Hochverfügbarkeit, Nodes sowie Upgrades und Backups verwaltet. |

## Was dich in diesem Kurs erwartet

Die Admin-Sektion richtet sich an alle, die einen Cluster nicht nur nutzen, sondern auch betreiben. Die Inhalte orientieren sich an den Themen der Certified Kubernetes Administrator (CKA) Prüfung.

- [01 Namespace](1-k8s-admin-namespace.md) — Cluster logisch aufteilen, Kontexte wechseln und ResourceQuotas setzen
- [02 User](2-k8s-admin-user.md) — Role Based Access Control mit Roles, ClusterRoles, Bindings und ServiceAccounts
- [03 Network](3-k8s-admin-network.md) — Netzwerkmodell, CNI, kube-proxy, CoreDNS und NetworkPolicies
- [04 High Availability](4-k8s-admin-highavailabity.md) — Control Plane, etcd und Worker Nodes ausfallsicher aufbauen
- [05 Nodes](5-k8s-admin-nodes.md) — Nodes registrieren, warten (cordon/drain) und Kapazitäten verstehen
- [06 Cluster Maintenance](6-k8s-admin-maintenance.md) — Upgrades mit kubeadm, etcd Backups, Zertifikate und Troubleshooting

## Voraussetzungen

- Sicherer Umgang mit den Themen aus den [Basics](../k8s-basic/0-k8s-basic-intro.md), insbesondere [Master](../k8s-basic/1-k8s-basic-master.md), [etcd](../k8s-basic/2-k8s-basic-etcd.md) und [Node](../k8s-basic/3-k8s-basic-node.md)
- Ein Cluster mit Admin-Rechten, idealerweise ein Multi-Node-Cluster (zum Beispiel mit kind oder kubeadm auf mehreren VMs), um Themen wie Node-Wartung und HA nachvollziehen zu können

## Tipps für die CKA-Prüfung

- Die Prüfung ist praktisch: Du arbeitest auf echten Clustern in einem Terminal. Übe deshalb vor allem auf der Kommandozeile.
- Nutze imperative Befehle mit `--dry-run=client -o yaml`, um YAML-Vorlagen schnell zu erzeugen, statt sie von Hand zu schreiben.
- Achte bei jeder Aufgabe auf den richtigen Kontext und Namespace (`kubectl config use-context ...`, `-n <namespace>`).
- `kubectl explain <resource>.<feld>` hilft, wenn dir ein Feldname nicht einfällt. Die offizielle Dokumentation unter [kubernetes.io/docs](https://kubernetes.io/docs/) darf in der Prüfung verwendet werden.
