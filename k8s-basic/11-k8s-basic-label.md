# Kubernetes basics: Labels

|||
|---|---|
| Title | K8 Basic Labels |
| Category | Course |
| Level | Novice |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-basic-training-etcd](https://muellermh.wordpress.com/k8s-basic-training-etcd)  |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/) |
| Description | In dieser Lektion lernst du was ein Label ist und wie diese selektiert werden können. |

## Labels

Labels sind Key Value Paare (key=value) die an REST Objekte in der Kubernetes API gehängt werden können die nicht für das System selbst relevant sind aber dem User das leben vereinfachen. Mit Hilfe dieser Key Value Paaren können Objekte gefiltert und zugeordnet werden. Labels können beim erstellen hinzugefügt werden und jederzeit hinzugefügt, verändert oder entfernt werden. Bei den Labels gibt es keine Begrenzungen und Sie sind frei wählbar. Jeder Key muss für ein Objekt eindeutig sein.

Ein paar Beispiel Labels, welche die Organisation von Objekten erleichtern:

```json
"release" : "stable",
"release" : "canary"
"environment" : "dev",
"environment" : "qa",
"environment" : "production"

"tier" : "frontend",
"tier" : "backend",
"tier" : "cache"

"partition" : "customerA",
"partition" : "customerB"

"track" : "daily",
"track" : "weekly"
```

## Label selectors

Labels können mit gleich oder ungleich selektiert werden, eine verknüpfung mehrere Tags wird mit einem Komma realisiert.

### Equality-based requirement

Gleichheits- oder ungleichheitsbezogene Selektierung ermöglichen die Filterung nach Labelschlüsseln und -werten. Abgleichobjekte müssen alle angegebenen Labelbeschränkungen erfüllen, können aber auch zusätzliche Labels haben. Es werden drei Arten von Operatoren zugelassen =,==,!=. Die ersten beiden repräsentieren Gleichheit (und sind einfach Synonyme), während die letzteren Ungleichheit darstellen.

```yaml
environment = production
tier != frontend
```

### Set-based requirement

Set-basierte Label Selektierung ermöglichen das Filtern von Schlüsseln nach einem Satz von Werten. Es werden drei Arten von Operatoren unterstützt: in, notin und exists (nur die Schlüsselkennung).

```yaml
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

Beispiel Befehl um eine Lister alle Pods mit Labels zu selektieren:

```shell
kubectl get pods -l 'environment in (production, qa)'
kubectl get pods -l 'environment in (production),tier in (frontend)'
 kubectl get pods -l 'environment,environment notin (frontend)'
 ```

In der Beschreibung von Objekten können diese Selektoren ebenso verwendet werden:

```yaml
selector:
    environment: production
```

oder komplexer:

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

> Labels können auch an Nodes gehängt werden, zum Beispiel um zu selektieren auf welchen Nodes ein Pod starten darf.