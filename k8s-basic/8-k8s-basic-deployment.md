# Kubernetes basics: Deployments

|||
|---|---|
| Title | K8 Basic training |
| Category | Course |
| Level | Novice |
| Duration | ? |
| Url | ... |
| Description | By the end of this course you will understand the basic topics of k8s. You will learn how container, pods, replica, services, secrets, labels and deployments works also with helath-checks.   |

## Deployment

A Deployment controller provides declarative updates for Pods and ReplicaSets.
You describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

## Typische UseCases

- Erstellen eines Deployments zum ausrollen eines ReplicaSets
- Updaten der Pods Spezifikation wobei ein neues ReplicaSet ausgerollt wird und alle alten Pods Replicas mit neuen ersetzt werden
- Rollback zu einer frühren Version
- Scalierung der Applikation
- Pausieren des Deplyoments umm mehrere Änderungen an der Pod Spezifizierung zu machen um dann ein neuen Rollout zu starten
- Den Status des Deployments nutzten um misserfolge beim Rollout zu sehen
- Bereinigung von alten ReplicaSets die nicht mehr benötigt werden

## Updaten eines Deployments

Für das aktuallisieren des Images reicht folgender Befehl
```bash
kubectl set image deployment/nginx nginx=nginx:1.15.1
```

Um das Deployment yaml zu bearbeiten benötigt man diesen Befehl

```bash
kubectl edit deployment/nginx
```

Um den rollout status zu prüfen reicht dieser Befehl:

```bash
kubectl rollout status deployment/nginx
```

um sich alle deployments anzuzeigen benötigt man diesen befehl

```bash
kubectl get deployments
```

```bash
kubectl get rs
```

 für die anzeige der ReplicaSet

```bash
kubectl get pods
```

für die anzeige der Pods

Sollte etwas nicht wie gewünscht funktionieren findet man alle informationen dazu im deployment und kann sich diese mit dem Befehl anzeigen lassen

```bash
kubectl describe deployments
```

Eine history des rollouts bekommt man mit

```bash
kubectl rollout history deployment/nginx
```

Ein Rollback kann man auch einfach mit dem Befehl ausführen

```bash
kubectl rollout undo deployment/nginx
```

Einfaches scallieren des Deployments mit

```bash
kubectl scale deployment nginx
```

Und um das Deployment zu pausieren einfach  

```bash
kubectl rollout pause deployment/nginx
```

um es anschließend mit

```bash
kubectl rollout resume deployment/nginx
```

weiter laufen zu lassen.

## Example

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: test-deployment
  name: test-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: test-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
```