# Kubernetes basics: Volumes

|||
|---|---|
| Title | K8 Basic Volumes |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | NA |
| Blog |   |
| Author | Alexander Giesa |
| Mail | ag@kubernauts.de |<S-F7><S-F6>
| Resource | [https://kubernetes.io/docs/concepts/storage/volumes/](https://kubernetes.io/docs/concepts/storage/volumes/)
|  |[https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) |
| Description |  |

## Volumes

Volumes sind Objekte, die zum persistieren von Daten dienen. 
Dateien in Containern sind temporär. Wird ein Container neu gestartet, sind alle Daten gelöscht. Damit die Daten auch nach einem Neustart erhalgen bleiben, werden Volumes genutzt.  
Es existiert noch ein weiterer Anwendungsfall. Volumes können auch genutzt werden, um Daten in einem Pod/Node/Cluster zwischen verschiedenen Containern zu teilen.

Volumes werden immer deklarativ erstellt. Daher muss eine YAML oder JSON Datei vorab erzeugt werden. YAML Dateien der Standard zur Konfiguration in der Kuberneteswelt sind.

Es gibt verschieden Arten von Volumes in Kubernetes.

- emptyDir
  - Wirkungsbreich
    Pod
  - Verwendungszweck  
    Wird üblicher Weise verwendet um in einem Pod Daten zwischen Containern austauschen oder gemeinsam zu nutzen. 
  - Lebenszyklus
    - Start  
      Das Volume wird erstellt, wenn ein Pod auf einem Node startet.  
    - Ende  
      Es ist ein temporärer Datentyp und wird gelöscht, sobald der Pod entfernt wird.  
  - Konfiguration  
    wird direkt in der Poddefinition gemountet.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pd
    spec:
      containers:
      - image: k8s.gcr.io/test-webserver
        name: test-container
        volumeMounts:
        - mountPath: /cache
          name: cache-volume
      volumes:
      - name: cache-volume
        emptyDir: {}
    ```

### Persistant Volume (PV)

- local
  - Wirkungsbereich
    Node
  - Verwendungszweck  
    Wird üblicher Weise verwendet um auf einem Node Daten zwischen Containern austauschen oder gemeinsam zu nutzen. 
  - Lebenszyklus
    - Start
      Das Volume kann erstellt werden, sobald der Ziel Node gestartet ist.  
    - Ende  
      Das Volume wird gelöscht, wenn der Node entfernt wird.
  - Konfiguration
    ```yaml
    kind: PersistentVolume
    apiVersion: v1
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

Weiter PV Typen

- GCEPersistentDisk
- AWSElasticBlockStore
- AzureFile
- AzureDisk
- CSI
- FC (Fibre Channel)
- FlexVolume
- Flocker
- NFS
- iSCSI
- RBD (Ceph Block Device)
- CephFS
- Cinder (OpenStack block storage)
- Glusterfs
- VsphereVolume
- Quobyte Volumes
- HostPath (Single node testing only – local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)
- Portworx Volumes
- ScaleIO Volumes
- StorageOS

### Persistant Volume Claim (PVC)

Persistant Volume Claim ist ein besonderes Volume. PVC´s werden verwendet, um Daten aus dem Persistant Volume (pv) in einen Pod zu Mounten. Ein PVC Speicher wird erst gültig nach der erfolgreichen Zuordnung eines PV´s. Es ist zu beachten, dass das zu zuordnende PV immer mindenstens ganau so viel Speicherplatz bereitstellen muss wie im PVC angefordert wird.

- persistentVolumeClaim
  - Wirkungsbereich
    Gesamtes Cluster
  - Verwendungszweck
    Mit PVC´s werden PV in Pods eingebunden, wie z.B. local Volumes. 
  - Lebenszyklus
    - Start  
      Mit dem ersten Node
    - Ende  
      Mit dem letzten Node
  - Konfiguration
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: myclaim
    spec:
      accessModes:
        - ReadWriteOnce
      volumeMode: Filesystem
      resources:
        requests:
          storage: 8Gi
      storageClassName: slow
      selector:
        matchLabels:
          release: "stable"
        matchExpressions:
          - {key: environment, operator: In, values: [dev]}
    ````

### Paramter
```yaml
- storageClassName: <Name>              # Name der Speichergruppe, über diesen Wert wird das Volume angesprochen/zugeordnet
- capacity: 
  - storage: <Ei, Pi, Ti, Gi, Mi, Ki>   # Speichergroesse z.B 5Gi um 5 Gigabyte zu reservieren
- accessMode: <ReadWriteOnce, ...>      # Zugriffskontrolle, sequenziell, parallel, ....
- hostPath: <Dateipfad>                 # Vereichnis in dem die Daten abgelegt werden, z.B. /mnt/shared
```