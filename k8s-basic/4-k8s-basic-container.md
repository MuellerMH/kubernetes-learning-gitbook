# Kubernetes basics: Container

|||
|---|---|
| Title | K8 Basic Container |
| Category | Course |
| Level | Novice |
| Duration | ? |
<<<<<<< HEAD
| Url | ... |
| Description | By the end of this course you will understand the basic topics of k8s. You will learn how container, pods, replica, services, secrets, labels and deployments works also with helath-checks.   |
=======
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/containers/images/](https://kubernetes.io/docs/concepts/containers/images/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Container und wie Kubernetes mit diese umgeht.  |
>>>>>>> 880b971e13c006e476ca8bb4614174c74d3281cd

---

## Images

[kubernetes concepts](https://kubernetes.io/docs/concepts/containers/images/)

In Kubernetes werden Container genau so behandelt wie in Docker.
Es wird ein Dockerfile erstellt, ein Image gebaut und in der Registry zur verfügung gestellt. Kubernetes kann mit den verschiedenen Repositorys arbeiten und auch wie gewohnt mit den verschiedenen Tags umgehene.

### Update LifeCycle

Um ein Image auf Kubernetes so einzurichten, dass es auf den Latest Tag reagieren kann mussen noch zusätzliche option hinzugefügt werden. Standard mäßig gilt die ImagePullPolicy: IfNotPresent. Sollte also das Image nicht existieren wird es gepullt. Um diese Standard Einstellung zu ändern muss folgendes gesetzt werden:

- imagePullPolicy: Always;

Wichtig ist wie bereits erwähnt das der Tag latest gewählt wurde.
Zusätzlich muss noch der AlwayPullImages admission Controller aktiviert werden.

Die Registry Zugangsdaten können mit kubectl über ein secret zur Verfügung gestellt werden. Dieses Thema wird später noch seperat in 07-k8-basic-secret behandelt.

```bash
kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
secret "myregistrykey" created.
```

### Pre-Pulling Images

Neben dem Pullen von Images wenn diese gebraucht werden gibt es noch den Workflow des Pre-Pulling. Hierbei werden alle nötigen Images auf alle Nodes vorab gepullt und zur Verfügung gestellt. Dies hat den Vorteil das keine Registry Credentials benötigt werden und Container schneller gestartet werden können da nicht mehr auf den Pull der Images gewartet werden muss.

### Local images

Sollen ausschließlich die Lokalen Images vom Node oder eines Netzwerk Laufwerkes zur Verfügung gestellten Images genutzt werden kann man dies lösen in dem man die imagePullPolicy auf **Never** stellt. Hierdurch werden keine Images mehr aus der Registry geladen.

---

## Environment variables

Kubernetes Container stellen verschiedene Environment Variablen zur verfügung.

Zum einen natürlich die selbst definierten Variablen im Dockerfile. Diese können mit Config-Maps und Secrets befüllt worden sein und werden in späteren Verlauf noch vertieft.

Zusätzlich stellen Kubernetes Container Informationen zum Hostnamen, Pod Namen und Namespace zur verfügung. Desweiteren stehen Custer Informationen zur verfügung und wenn das DNS Add-on aktiviert ist auch die IP Adressen der laufenden Container zum Zeitpunkt der erstellung.

---

## Lifecycle Hooks

In Kubernetes gibt die Lifecycle hooks die von den Containern zur verfügung stehen, welche implementierte Handler ausführen wenn ein Hook aufgerufen wurde.

### Container Hooks

Es gibt zwei Hooks die von Containern zur verfügung gestellt werden. Zum einen der **PostStart**, der immer ausgeführt wird nachdem ein Container erstellt worden ist. Der Zweite Hook ist der **PreStop**, der immer ausgeführt wird bevor ein Container terminiert wird. Dieser Hook blockiert den Prozess der terminierung bis dieser abgearbeitet ist.

### Hooks implementieren

Hooks können auf zwei verschiedene Weisen implementiert werden. Zum einen über Exec welche als shell script zur verfügung gestellt werden, zum anderen als Http Call, welcher einen spezifischen Endpunkt auf dem Container aufruft.

### Hooks execution

Wenn ein Container lifecycle Hook angesteuert wird führt das Kubernetes manager system die ausführung des registrierten Handlers durch. Hooks werden immer synchron aufgerufen und der weitere Prozess des Containers blockiert bis der Hook abgeschlossen ist.

### Hook delivery guarantees

Hooks können mehrmals aufgerufen werden, daher muss in der Implementierung damit umgegangen werden.

### Hook debugging

Hook logs werden nicht an die Pod Events weitergeleitet, sollte ein Fehler auftreten werden diese über ein broadcast event weitergeleitet. Diese Events können über `kubectl describe pod` eingesehen werden.