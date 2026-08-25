# Kubernetes advance: ConfigMaps

|||
|---|---|
| Title | K8 Advance ConfigMaps |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-advance-training-configmap](https://muellermh.wordpress.com/k8s-advance-training-configmap) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/configuration/configmap/](https://kubernetes.io/docs/concepts/configuration/configmap/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema ConfigMap |

## ConfigMap

Eine ConfigMap ist ein Kubernetes Objekt, das nicht vertrauliche Konfigurationsdaten als Key-Value Paare vorhält. Damit lässt sich die Konfiguration einer Anwendung vom Container Image trennen, so dass dasselbe Image in unterschiedlichen Umgebungen (dev, staging, prod) mit jeweils eigener Konfiguration laufen kann.

Eine ConfigMap ist nicht für sensible Daten wie Passwörter oder Zertifikate gedacht, dafür gibt es das [Secret](../k8s-basic/12-k8s-basic-secret.md). Der Inhalt einer ConfigMap wird nicht verschlüsselt abgelegt.

## ConfigMap erstellen

Wie bei den meisten Kubernetes Objekten gibt es einen imperativen und einen deklarativen Weg, eine ConfigMap zu erstellen.

### Imperativ mit kubectl

Über `--from-literal` lassen sich einzelne Key-Value Paare direkt auf der Kommandozeile setzen:

```shell
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=production
```

Über `--from-file` wird stattdessen der Inhalt einer Datei als Wert übernommen. Der Dateiname wird dabei zum Key:

```shell
kubectl create configmap app-config-file --from-file=./application.properties
```

### Deklarativ mit YAML

Alternativ lässt sich eine ConfigMap wie jedes andere Objekt als YAML beschreiben und mit `kubectl apply -f` erstellen:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: production
  application.properties: |
    server.port=8080
    logging.level=INFO
```

Der Key `application.properties` zeigt, dass in einer ConfigMap auch ganze Dateien als mehrzeiliger Wert abgelegt werden können.

## ConfigMap verwenden

### Als Environment Variable

Eine ConfigMap kann Pods als Environment Variablen zur Verfügung gestellt werden:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: APP_COLOR
```

Damit steht die env Variable `$APP_COLOR` im Container zur Verfügung. Soll die komplette ConfigMap als Environment Variablen übernommen werden, geht das kompakter mit `envFrom`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-envfrom-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    envFrom:
      - configMapRef:
          name: app-config
```

### Als Volume

Genau wie ein Secret kann eine ConfigMap auch als Volume gemountet werden. Jeder Key wird dabei zu einer Datei im Mount Pfad, der Wert ist der Dateiinhalt:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: "/etc/config"
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

Die Datei `/etc/config/application.properties` enthält damit den Inhalt des gleichnamigen Keys aus der ConfigMap. Wird die ConfigMap aktualisiert, werden gemountete Dateien automatisch nachgezogen, allerdings mit einer Verzögerung durch die Synchronisierung des Kubelet. Über Environment Variablen bezogene Werte werden dagegen erst nach einem Neustart des Pods aktualisiert.
