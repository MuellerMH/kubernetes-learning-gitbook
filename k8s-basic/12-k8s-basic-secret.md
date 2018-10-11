# Kubernetes basics: Secrets

|||
|---|---|
| Title | K8 Basic Secrets |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description |  |

## Secrets

Secrets sind Objekte die sensible Daten vorhalten und können als Volume, als Enviroment Variablen an einen Pod gehängt werden. Ebenso können Secrets von der Kubectl genutzt werden um Beispielsweise Docker Hub Images mit bestimten Zugangsdaten abzurufen.
Dies hat den Vorteil, dass diese sensiblen Daten nicht im Docker Image oder in der Pod Description hinterlegt werden müssen. Zudem sind diese Daten leicht aktuallisierbar und gelten somit für alle angehängten Pods. Hierduch lassen sich Problemlos stärkere Regelen für Passwörter und Sicherheits relevante Informationen einsetzten, wie Beispielsweise die regelmäßige aktuallisierung von Passwörtern.

Secrets werden nicht nur vom User angelegt, auch Kubernetes selbst kann Secrets anlegen.

### Build-in Secrets

Kubernetes erstell automatisch Secrets mit API Zugangsdaten und passt die Pods automatisch an diese zu nutzten.
Diese funktion kann auch ausgestellt oder überschrieben werden.

## Eigene Secrets erstellen

Meist werden Secrets erstell wenn Pods Zugangsdaten für dritte System braucht, als Beispiel eine Datenbank.
Die Daten für die Secrets können entweder direkt im kubectl Befehl eingegen werden oder über Files hinzugefügt werden.
Nutzername und Passwörter müssen hierbei als Base64 String encoded sein.

Dies ist jedoch nicht zwingend erforderlich, da Kubernetes diese mit dem zusatz generic selbst decoden kann.

```shell
## lokale vorbereitung
echo -n 'admin' > ./username.txt # admin in die username.txt schreiben
echo -n "12345Passwort" > ./password.txt # passwort in die password.txt schreiben
## erstellen des Secrets
kubectl create secret generic mysecret --from-file=./username.txt --from-file=./password.txt
```

Ein Yaml fiel muss bereits base64 encoded Daten enhalten und sieht wie folgt aus:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

In der console generierst du den base64 String am einfachsten mit

```shell
echo -n "admin" | base64
```

Diese Yaml kann mit dem üblichen kubectl Befehl erstellt werden

```shell
kubectl create -f ./secret.yaml
```

Naturlich kann ein Secret auch jederzeit mit der kubectl ausgelesen und als yaml file abgelegt werden

```shell
kubectl get secret mysecret -o yaml
```

Um anschließend den Klartext lesen zu können muss einfach der String mit base64 decoded werden
In der Shell sieht das wie folgt aus:

```shell
echo "YWRtaW4=" | base64 --decode
```

## Secrets verwenden

### als Volume

Ein Secret kann mit dem zusatz der Volume angabe in einem Yaml definiert werden. Hierfür muss das Secret jedem Container der darauf zu greifen können soll als Volume mitgeben werden. Das Volume ist dann unter im container definierten Pfad verfügbar hier im Beispiel ./etc/foo/username und ./etc/foo/password

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

Natürlich können auch Key spezifische Pfade angebene werden. Hier als Beispiel wird der username under dem Pfad /etc/foo/my-group/my-username abgelegt. Der Key Passwort wird hier im Beispiel ignoriert und nicht zur verfügung gestellt.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 511
```

User Rechte können mit dem Key `mode:` noch gesetzt werden.
Die gemounteten Secretes werden autmoatisch aktuallisiert, wenn sich das Secret ändert.

### Als Environment Variables

In vielen fällen möchte man die Secrets jedoch als Environment Variablen zur Verfügung haben, zum Beispiel in einer Java Spring Boot Applikation. Das ist natürlich auch möglich in dem du statt des volumes einfach env: definierst.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
```

Damit sind die env Variablen $SECRET_USERNAME und SECRET_PASSWORD im definierten Container verfügbar.

### Docker Registry

Wie einleitend beschrieben können Secrets auch für die Docker Registrie verwendet werden. Dazu erstellt man ein neues secret mit dem zusatz docker-registry:

```shell
kubectl create secret docker-registry myregistrykey --docker-server=https://hub.docker.com --docker-username=myusername --docker-password=secretpassword
```

Alternative kann dies natürlich auch mit einer yaml File beschrieben werden:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
  namespace: awesomeapps
data:
  .dockerconfigjson: UmVhbGx5IHJlYWxseSByZWVlZWVlZWVlZWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGx5eXl5eXl5eXl5eXl5eXl5eXl5eSBsbGxsbGxsbGxsbGxsbG9vb29vb29vb29vb29vb29vb29vb29vb29vb25ubm5ubm5ubm5ubm5ubm5ubm5ubm5ubmdnZ2dnZ2dnZ2dnZ2dnZ2dnZ2cgYXV0aCBrZXlzCg==
type: kubernetes.io/dockerconfigjson
```

Hierbei musst du vorab deine .docker/config.json base64 encode und dann als data.dockerconfigjson übergeben.
Dem Container wird dann das imagePullSecret, so wie dieser type von Secret heißt, an den Container gehängt.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
```