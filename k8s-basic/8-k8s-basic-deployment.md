# Kubernetes basics: Deployments

|||
|---|---|
| Title | K8 Basic Deployments |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Deployment |

## Deployment

Ein Deployment Controller stellt deklarative Updates für Pods und ReplicaSets zur Verfügung.
Sie beschreiben einen gewünschten Zustand in einem Deployment-Objekt, und der Deployment Controller ändert den aktuellen Zustand mit einer kontrollierten Geschwindigkeit in den gewünschten Zustand. Sie können Deployments definieren, um neue ReplicaSets zu erstellen, oder um bestehende Deployments zu entfernen und alle ihre Ressourcen mit neuen Deployments zu übernehmen.

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