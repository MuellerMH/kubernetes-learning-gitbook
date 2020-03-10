# Kubernetes basics:  Nodes

|||
|---|---|
| Title | K8 Basic Nodes |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/architecture/nodes/](https://kubernetes.io/docs/concepts/architecture/nodes/) |
| Description | Am Ende dieses Kurses werden Sie die grundlegenden Themen der k8s Nodes verstehen. |

---

## What is a node

Ein Node ist eine physische oder virtuelle Servereinheit die dem Cluster als Worker Maschine dient. Allgemein wird sie als "minion" bezeichnet und verfügt über einige Komponenten die für das Kubernetes Cluster benötigt werden.
Der Server kann entweder ein dedizierter Server im eigenen Rechenzentrum sein, oder eine virtuelle Instance eines Servers auf einem Host System. Natürlich können die Instancen auch in den verschiedenen Public Clouds wie AWS, Google Cloud, Azure oder in Private Clouds wie OpenStack laufen.

Eine Node beinhaltet immer die grundlegenden Komponenten um Pods laufen zu lassen und wird von der Master Komponenten des Clusters gemanaged. Grundlegende Komponenten sind eine Container Laufzeitumgebung wie zum Beispiel Docker, Kubelet und Kube-Proxy.

### Docker

Docker ist eine offene Plattform für die Entwicklung, den Versand und die Ausführung von Anwendungen.

### Kubelet

Kubelet ist ein Node Agent, ein RestAPI EntryPoint, der auf jeder Node laufen muss. Der Agent nimmt die Befehle vom Master in Form einer Yaml oder Json defeinitionen von Pods und sorgt dafür das diese ausgeführt werden und überwacht ebenso das diese Pods laufen und gesund sind.

### Kube-Proxy

Kube-Proxy ist der Kubernetes Network Proxy und muss ebenso auf allen Nodes laufen. Dem Proxy sind alle definierten Service bekannt und ist in der Lage die Verbindungen über TCP und UDP auf einen oder mehrere adressierte Pods zu verteilen.

---

## Node status

Der Node Status beinhaltet verschiedene Informationen über sich selbst. Diese teilen sich in 4 Unterkategorien auf.

### Addresses

Der Node Status beinhaltet im Addressen Bereich sein:

- HostName
- Seine Externe IP Adresse
- Seine Interne IP Adresse

### Condition

Der Node Status liefert in der Kategorie Condition seinen aktuellen zustand.

- OutOfDisk (true/false) wenn dieser wert true ist hat die Node nur noch wenig freien Festplatten speicher
- MemoryPressure (true/false) wenn dieser wert true ist hat die Node nur noch wenig freien Arbeitsspeicher
- PIDPressure (true/false) wenn dieser wert true ist laufen zu viele Processe auf der Node
- DiskPressure (true/false) wenn dieser wert true ist hat die Node nur noch wenig freien Festplatten speicher
- NetworkUnavaible (true/false) wenn dieser wert true ist hat die Node kein Netzwerk oder eine incorrecte Konfiguration des Netzwerkes und ist somit nicht erreichbar
- ConfigOK (true/false) wenn dieser wert false ist dann ist kublet nicht richtig konfiguriert

Zusätzlich gibt es noch die Confition *READY* diese kann 3 verschiedene Informationen enthalten.

- True = die Node ist okay und alles arbeitet wie erwartet
- False = das management des Nodes funktioniert nicht wie erwartet
- Unknown

### Capacity

Die kapazität beinhaltet Information zur auslastung der Node. Hier wird die CPU sowie Memory Auslastung und Verfügbarkeit geliefert und zusätzlich die Zahl der maxiamlen Anzahl von Pods die auf der Node gestartet werden können.

### Info

Die Info beinhaltet allgemeine Informationen zur Node, darunter fallen die Kernel Version, die Kubernetes Version ( kubelet und kube-Proxy), die Docker Version und der Name des Operation Systems (OS)

---

## Node managment

Im Gegensatz zu Pods und Services werden die Nodes nicht von Kubernetes selbst erstellt. Die bereitstellung von Physischen oder Virtuellen Servern erfolgt durch den Provider. Zum Beispiel über die Bereitstellung eines neuen Servers im Rechenzentrum oder das provisioneren einer Virtuellen Maschine über die Public/Private Cloud. Wenn in Kubernetes eine Node angelegt wird, ist dieses nur ein definiertes Objekt, welches eine Node repräsentiert. Kubernetes prüft ob die definierte Node valide ist. Sollte dies der Fall sein wird sie im Cluster verwendet um Pods und Services bereit zu stellen. Kubernetes behält auch die nicht validen definierten Objekte für Nodes und prüft regelmäßig ob diese mittlerweile valide sind.

---

## Node controller

Der Node Controller ist eine Master Komponente die zum managen der Node benötigt wird.
Als erstes ist der Node Controller verantwortlich ein CIDR Block zu vergeben wenn die Node registriert wird.
Desweiteren hält der Node Controller eine Liste der verfügbaren Nodes und aktuallisiert diese ständig und gleicht sie mit den verfügbaren Maschinen der Providers ab. Wenn eine Node nicht mehr ordentlich arbeit wird abgeglichen ob die Maschiene noch beim Provider existiert. Sollte die Maschiene nicht mehr existieren wird der Node controller diese von der Liste streichen.
Die dritte Aufgabe des Node Controllers ist die Verfügbarkeit der Nodes zu kontrollieren. Sollte dieser nicht mehr erreichbar sein werden nach einer weiteren prüfung alle Pods und Service vom defekten Node entfernt.

Folgende Cloud Provider werden von Kubelet nativ unterstüzt:

- AWS
- Azure
- GCE
- CloudStack
- OpenStack
- OVirt
- Photon
- VSphere
- IBM Cloud Kubernetes Sevice