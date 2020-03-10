
---
marp: true
---

# Kubernetes Basic:  Master

|||
|---|---|
| Title | K8 Basic Master |
| Category | Course |
| Level | Novice |
| Duration | 3:46 |
| YouTube | [Basic Master](https://www.youtube.com/watch?v=_RePboiZnJY) |
| Blog | https://muellermh.wordpress.com/k8s-basic-training-master  |
| Author | Manuel H. Müller alias Onko |
| Mail | mm@kubernauts.de |
| Resource | https://kubernetes.io/docs/concepts/architecture/nodes/ | 
| Description | By the end of this course you will understand the basic topics of k8s nodes. What is a node, how kubernetes controll the nodes and how to self-register a node to the cluster |

---

## Master

Der Master besteht aus mehreren Komponenten. Diese treffen globale Entscheidungen über den Cluster, wo zum Beispiel welche Pods gestarted werden und ekennt und reagiert auf Cluster Ereignisse. Hierunter fallen startende und ausgefallene Pods oder wenn die Replikationen nicht erfüllt sind. Theoretisch können die Master Komponenten mit auf den Nodes laufen, jedoch hat es sich in der Praxis bewährt diese auf zwei eigene Instanzen auszulagern. Es wurden zwei Instanzen gewählt um den Single Point of Fail zu lösen der bei einer einzelnen Instanze zu tragen käme. Mehr als zwei Master Instanzen bieten jedoch keinen nennenswerten Vorteil.

---

### Kube-API server°
=======
| Resource | [Link to resources](https://kubernetes.io/docs/concepts/overview/components/) |
| Description | Am Ende dieser Lektion wirst du die grundlegenden Themen der k8s Master verstehen.|

---

## Master Componenten

--

Der Master besteht aus mehreren Komponenten. Diese treffen globale Entscheidungen über den Cluster, wo zum Beispiel welche Pods gestarted werden und ekennt und reagiert auf Cluster Ereignisse.

--

Hierunter fallen startende und ausgefallene Pods oder wenn die Replikationen nicht erfüllt sind. Theoretisch können die Master Komponenten mit auf den Nodes laufen, jedoch hat es sich in der Praxis bewährt diese auf zwei eigene Instanzen auszulagern.

--

Es wurden zwei Instanzen gewählt um den Single Point of Fail zu lösen der bei einer einzelnen Instanze zu tragen käme. Mehr als zwei Master Instanzen bieten jedoch keinen nennenswerten Vorteil.

---

### Kube-API server

--
>>>>>>> 880b971e13c006e476ca8bb4614174c74d3281cd

Diese Komponente stellt die Kubernetes API zur verfügung. Die API ist der dreh und ankerpunkt des Kubernetes Clusters. Sie ist horizont scalierbar ausgerichtet und somit stateless.

---

### Kube-scheduler

Diese Komponente überwacht die neu gestartete Pods die noch keinem Node zugeordnet sind. Prüft auf welcher Node die Resourcen für den Pod vorhanden sind und gibt dem Node über das kubelet die Informationen um den Pod zu managen.

---

### Kube-controller-manager

Diese Komponente kümmert sich aussschließlich um Kubernetes eigenen Controller. Darunter fallen die Node Controller, die Replication Controller, die Endpoint Controller sowie die Service, Account und Token Controller.

---

### Cloud-controller-manager

Diese Komponente ist die Schnittstelle zu dem unterliegenden Cloud Provider. Hier werden gekapselt die Funktionen des Cloud Providers zur verfügung gestellt und von folgenden Controllern genutzt:

---

- Node Controller um zu prüfen ob die darunter liegende Instance beim Provider noch existiert
- Route Controller für das setzten der routen in der unterliegenden Cloud Infrastruktur
- Service Controller für das erstellen, updaten und entfernen von Load Balancern
- Volume Controller für das erstellen, anhängen, updaten und entfernen von Datenspeichern

---

Folgende Cloud Provider werden von Kubernetes aktuell unterstüzt, für die Entwicklung sind, seit der Version 1.6,die Cloud Provider selbst verantwortlich.

---

- AWS
- Azure
- GCE
- CloudStack
- OpenStack
- OVirt
- Photon
- VSphere
- IBM Cloud Kubernetes Sevice