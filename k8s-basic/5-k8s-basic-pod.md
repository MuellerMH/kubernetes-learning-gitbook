# Kubernetes basics:  Pods

|||
|---|---|
| Title | K8 Basic Pods |
| Category | Course |
| Level | Novice |
| Duration | ? |
| Url | ... |
| Description | By the end of this course you will understand the basic topics of k8s. You will learn how container, pods, replica, services, secrets, labels and deployments works also with helath-checks.   |

---

## Pod allgemein

Ein Pod ist eine Gruppe von einem oder mehreren Containern. Hierbei sollte die Skalibarkeit der Container innerhalb eines Pods beachtet werden. Es sollten Grundsätzlich nur Container zusammen in einem Pod gestarted werden, bei denen es eine entsprechende Abhängigkeit gibt.
Statt das Backend und das Frontend einer Applikation in einen Pod zu starten sollten diese zur skalierbarkeit in zwei verschiednen Pods laufen. Ein Container der die Logs einsammelt und die entsprechend weiterleitet kann natürlich im selben Pod gestartet werden.

### Managment

Container in einem Pod werden immer zusammen deployed, gestartet und gestopped. Ebenso wird die Replikation auf Pod Ebene geführt.

### Resourcen

Ein Pod ist ein logischer Host für die Container, alle Container teilen sich diesen und können über localhost miteinander kommunizieren. Alle Applikationen in einem Pod haben den gleichen Netzwerk Namespace, die gleiche IP und den gleichen Port space.

### Verwendung

Im vergeleich zu Docker entspricht ein Pod einem Docker-compose File. Geteile Volumes in einem Pod können von allen Containern genutzt werden.

Pods werden in der Regel genutzt um ein Support für co-located und co-managed Hilfs programme zu gruppieren.

Zum Beispiel ein Content Managment System mit Files, Datenbank, Cache und mehr.

[Original](https://kubernetes.io/docs/concepts/workloads/pods/pod/)

```text
content management systems, file and data loaders, local cache managers, etc.
log and checkpoint backup, compression, rotation, snapshotting, etc.
data change watchers, log tailers, logging and monitoring adapters, event publishers, etc.
proxies, bridges, and adapters
controllers, managers, configurators, and updaters
```

### Lebenscyclus

Pod dürfen als nicht dauerhafte Einheit gesehen werden. Die Pods können aus unterschiedlichsten Gründe ausfallen werden dann irgendwo neu erzeugt.

### Beenden eines Pods

Da die Pods auf verschiedenen Nodes laufen muss immer sicher gestellt werden das Pods sauber aufgeräumt werden. Daher werden erst alle Container angewisen sich zu terminieren, nach einer kurzen wartezeit werden alle Conatainer Prozesse gekillt. Sollte ein verwaltender Prozess wärend des beenden nicht verfügbar sein wird der beenden Prozess erneut ausgeführt.

Der Befehl dafür lautet `kubectl delete ...` dieser Befehl kann mit Flags erweitert werden. Zum Beispiel kann die Zeit zwischen Terminieren und Killen mit `--grace-period=<seconds>` gesetzt werden oder ein Force delete angefordert werden mit `--force`.

### Privilegierte Pods

Jeder Container in einem Pod kann im privilegierten Modus gestartet werden. hier für wird das Flag `--privileged` gesetzt. Damit hat der Container nahzu die selben Rechte wie der Host User auf dem er läuft.

---

## Lifecycle

### Pod Phase

Ein Pod kann sich in verschiedenen Phasen befinden, eine Liste im Überblick:

Wert | Beschreibung
--- | ---
Pending | der Pod befindet sich im Aufbau
Running | der Pod ist aktiv und läuft
Suceedded | alle container im Pod wurden beendet und nicht neu gestarted
Failed | alle container im Pod sind beendet und der letzte container ist in ein Fehler gelaufen
Unknown | Manchmal kann ein Pod aus ungeklärten Gründen in diesem Status stecken da vermutlich etwas mit der Kommunikation nicht stimmte

### Pod Zustände

Zusätzlich hat jeder Pod noch definierte Zustände in einem Array
 Wert | Beschreibung
 --- | ---
lastProbeTime | timestamp der letzten prüfung
lastTransisitionTime | timestamp der letzten Veränderung des Pods
message | eine lesbare Nachricht mit details zur Veränderung
reason | a unique, one-word, camelCase Reason for transistion
status | true false and unkown
type | transition type

Werte für den Typen von Transitions

Wert | Beschreibung
--- | ---
PodScheduled | Pod ist für ein node geplant
Ready | der Pod ist einsatzbereit
Initialized | alle container sind initalisiert und starten
Unschedulable | der Pod kann nicht geplant werden
ContainersReady | alle Container im Pod sind fertig

### Container Proben

Eine Probe ist eine rythmische Prüfung durch den Kubelet, der einen Handler im Container aufruft ob ein Container noch arbeitet. Es gibt 3 Arten von Handlern:

Wert | Beschreibung
--- | ---
ExecAction | führt ein Befehl im Container aus
TCPSocketAction | prüft anhand der IP ob der Container arbeitet
HTTPGetAction | prüft anhand eines WebRequest ob der Container noch reagiert

Jede Probe kann einen der Werte zurückliefern: `Success , Failure , Unkown`
Zusätzlich kan Kubelet noch zwei optionale Aktionen auf laufende Container durchführen.
`livenessProbe` welcher prüft ob der Container noch läuft und `readinessProbe` welcher prüft ob der Container noch Service Anfragen beantworten kann