# Kubernetes basics: Replication Controller

|||
|---|---|
| Title | K8 Basic Replication Controller |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description |  |

## Replication Controller

Der Replication Controller ist noch aus den Anfangen der Kubernetes Entwicklung. Es wird empfohlen Deployments und ReplicationSets stattdessen zu nutzen.

Der Replication Controller ist dafür zuständif eine spezifische Anzahl des gleichen Pods laufen zu lassen und wenn nötige anzupassen.

Generelle Kubernetes Config für ReplicationController:

```yaml
apiVersion: v1
kind: ReplicationControllerhttps://github.com/kubernauts/tk8/issues/34
metadata:
  name: nginx
```

### Pod Template

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:  
  template: # Pod Template
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
   - containerPort: 80
```

### Labels

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:  
  template:
    metadata:
      name: nginx
      labels: # Labels
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
   - containerPort: 80
```

### Pod Selector

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  selector: # Selector
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### Multiple Replicas

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3 # replicas
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

## How it works

Erstellen des Replication Controllers
`kubectl create -f replicationcontroller.yaml`

Überprüfung der Beschreibung des Replication Controllers
`kubectl describe replicationcontrollers/nginx`

Auflisten der Pods mit Selector
`kubectl get pods --selector=app=nginx`

## Working with Replication Controller

### Delete Replication controller and Pods

Mit dem Befehl `kubectl delete replicationcontroller/nginx` werden die Pods auf 0 scalliert und anschließend der Controller entfernt.

### Delete Replication controller only

Mit dem Befehl `kubectl delete replicationcontroller/nginx --cascade=false` werden die Pods nicht runter scalliert und nur der ReplicationController entfernt.

### Isolating pods from a replication controller

Manchmal ist es Notwendig einen Pod aus dem Replication Controller zu entfernen um diesen zu debuggen oder daten zu extrarieren. Ein Pod der aus dem ReplicationController entfernt wird, wird automatisch vom Controller nach scalliert.

Um einen Pod aus dem Replication Controller zu entfernen muss einfach das Label geändert werden. In diesem Beispiel wäre es das Label *app* von -nginx- in -debug- zu ändern.

`kubectl label pods NAME_DES_PODS app=debug`

## common usage

### Rescheduling

In erster Linie dient der ReplicationController dazu einen Pod zu reschedulen. Das heißt du möchtest 12 Replicas von diesem Pod laufen lassen und der Controller kümmert sich darum das immer 12 Replicas des Pods laufen.

### Scaling

Zudem erleichter der ReplicationController das scallieren. Da hier einfach ein neuer Wert gesetzt werden kann und der Replication Controller beginnt direkt mit seiner Arbeit und scalliert die Replicas des Pods auf die gewünschte Anzahl.

### Rolling updates

Ein weiterer Einsatz ist im Bereich des Rolling Updates vorhanden. Der Replication Controller wird nach und nach manuell um ein Replica verringert. Parallel dazu wird ein neuer ReplicaController gestarted bei dem nach und nach ein Replica hinzugefügt wird. Dadurch scalliert der alte ReplicationController sein Pods runter und der neue Replication Controller scalliert seine Pods hoch.

Das Rolling Update ist in der CLI implementiert, siehe hierzu [`kubectl rolling-update`](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/)

### multiple release tracks

Ebenso ist es möglich mit weiteren ReplicationControllern verschiedene Versionen parallel zu scallieren und diese über den gleichen Service zur verfügung zu haben. Als Beispiel könnten hier die Labels unterschiedlich gesetzt werden um zu schauen ob die Application bereits mit der latest version funktioniert oder sonstige informationen zu sammeln

```yaml
app: nginx
enviroment: prod
track: latest
```

und 

```yaml
app: nginx
enviroment: prod
track: stable
```

### A / B Testing

Dieser multiple release track ermöglicht es zudem A/B Testing zu fahren und verschiedene Version Round robin an die User auszuliefern.

## Writing programs for Replication

Pods die mit einem ReplicationController betrieben werden sollten ersetzbar und semantisch identisch sein. Die scallierten Applikationen sind im Idealfall stateless. Zum beispiel kann ein ReplicationController eine gewisses Set an Mastern die eine RabbitMQ befüllen bereitstellen während ein weiterer ReplicationController dafür sorgt das die entsprechenden Worker bereit stehen.

## Alternativen zum ReplicationController

- [ReplicaSet](7-k8s-basic-replica-set.md)
- [Deployment](8-k8s-basic-deployment.md)
- [Bare Pods](14-k8s-basic-bare-pod.md)
- [Job](12-k8s-basic-job.md)
- [DeamonSet](13-k8s-basic-daemonset.md)