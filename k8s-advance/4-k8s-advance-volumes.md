# Kubernetes advance: Volumes

|||
|---|---|
| Title | K8 Advance Volumes |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-advance-training-volumes](https://muellermh.wordpress.com/k8s-advance-training-volumes) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/storage/volumes/](https://kubernetes.io/docs/concepts/storage/volumes/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Volumes |

## Volumes

Dateien in Containern sind flüchtig. Wird ein Container neu gestartet, sind alle Daten, die im Container Dateisystem lagen, verloren. Damit Daten auch über einen Neustart hinweg erhalten bleiben, werden Volumes genutzt. Ein weiterer Anwendungsfall für Volumes ist das Teilen von Daten zwischen mehreren Containern innerhalb desselben Pods.

Volumes werden immer deklarativ per YAML oder JSON beschrieben. Es gibt in Kubernetes eine ganze Reihe unterschiedlicher Volume Typen, im Folgenden zwei der gebräuchlichsten.

### emptyDir

* Wirkungsbereich: Pod
* Verwendungszweck: Daten zwischen mehreren Containern innerhalb desselben Pods austauschen oder gemeinsam nutzen
* Lebenszyklus: Wird erstellt, sobald der Pod auf einem Node startet, und gelöscht, sobald der Pod entfernt wird. `emptyDir` ist damit rein temporär und nicht für dauerhafte Persistenz geeignet.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### hostPath

* Wirkungsbereich: Node
* Verwendungszweck: Eine Datei oder ein Verzeichnis vom Dateisystem des Nodes direkt in den Pod mounten, zum Beispiel für Werkzeuge, die auf Node-lokale Ressourcen zugreifen müssen
* Lebenszyklus: An den Node gebunden, nicht an den Pod. Der Inhalt bleibt erhalten, solange der Node existiert, wechselt der Pod jedoch den Node, ist der `hostPath` Inhalt des vorigen Nodes nicht mehr erreichbar. Für produktive Multi-Node Cluster ist `hostPath` deshalb nur eingeschränkt geeignet.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - image: nginx
    name: test-container
    volumeMounts:
    - mountPath: /test-data
      name: node-data
  volumes:
  - name: node-data
    hostPath:
      path: /data
      type: Directory
```

## Volume vs. PersistentVolume

Die bisher gezeigten Volume Typen (`emptyDir`, `hostPath`) werden direkt in der Pod Definition beschrieben. Ihr Lebenszyklus ist an den Pod oder den Node gekoppelt, es gibt kein eigenständiges Kubernetes Objekt dafür.

Ein **PersistentVolume (PV)** ist demgegenüber ein eigenständiges Cluster-Objekt für Speicher, dessen Lebenszyklus unabhängig von einem einzelnen Pod ist. Ein PV wird entweder vom Administrator vorab angelegt oder dynamisch über eine StorageClass provisioniert und existiert weiter, auch wenn der Pod, der ihn nutzt, längst gelöscht wurde.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: localdisk
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Damit ein Pod ein PersistentVolume nutzen kann, braucht er eine **PersistentVolumeClaim (PVC)**. Die PVC ist die Anforderung eines Pods an Speicher (Größe, Zugriffsmodus), die von Kubernetes einem passenden PV zugeordnet wird. Ein PVC wird erst gültig, sobald ihm ein PV zugeordnet ist, und das zugeordnete PV muss mindestens so viel Speicher bereitstellen, wie im PVC angefordert wurde.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: localdisk
```

Der PVC wird dann im Pod wie jedes andere Volume gemountet:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - image: nginx
    name: test-container
    volumeMounts:
    - mountPath: /data
      name: persistent-storage
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: myclaim
```

Kurz zusammengefasst: Ein einfaches Volume (`emptyDir`, `hostPath`) ist an Pod oder Node gebunden und verschwindet mit diesen. Ein PersistentVolume ist ein eigenständiges, vom Pod-Lebenszyklus entkoppeltes Speicherobjekt, das über eine PersistentVolumeClaim in einen Pod eingebunden wird.
