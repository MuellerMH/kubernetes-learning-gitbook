# Kubernetes admin: Namespace

|||
|---|---|
| Title | K8 Admin Namespace |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-admin-training-namespace](https://muellermh.wordpress.com/k8s-admin-training-namespace) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Namespace |

## Namespace

Ein Namespace ist eine logische Unterteilung eines Clusters. Er trennt Ressourcen wie Pods, Deployments oder Services voneinander, damit mehrere Teams oder Projekte denselben Cluster nutzen können, ohne sich gegenseitig mit Namen zu überschneiden.

Ein Namespace ist für sich genommen **keine Security-Boundary**. Ohne zusätzliche Maßnahmen wie RBAC oder NetworkPolicy können Pods aus unterschiedlichen Namespaces weiterhin miteinander kommunizieren, und ein Nutzer mit clusterweiten Rechten sieht alle Namespaces gleichermaßen. Der Namespace ordnet, er isoliert nicht automatisch ab.

## Default Namespaces

Ein frisch installierter Cluster bringt vier Namespaces mit:

* `default` — Namespace für Objekte, die ohne explizite Angabe eines Namespace angelegt werden
* `kube-system` — Objekte, die zur Kubernetes-Infrastruktur selbst gehören (z. B. CoreDNS, kube-proxy)
* `kube-public` — für alle Nutzer lesbar, auch unauthentifiziert; enthält meist clusterweit sichtbare Informationen
* `kube-node-lease` — enthält Lease-Objekte, über die Nodes ihren Heartbeat an den API-Server melden

```shell
kubectl get namespaces
```

## Namespaced vs. Cluster-Scoped Ressourcen

Nicht jede Ressource gehört zu einem Namespace. Die meisten Objekte (Pods, Deployments, Services, ConfigMaps, ...) sind **namespaced**: derselbe Name kann in unterschiedlichen Namespaces mehrfach vorkommen. Ein paar Ressourcen sind dagegen **cluster-scoped** und existieren nur einmal für den gesamten Cluster, unter anderem:

* Node
* PersistentVolume
* Namespace selbst
* ClusterRole / ClusterRoleBinding

> Rollen (`Role`, `RoleBinding`) sind dagegen namespaced. Details dazu behandelt [Kapitel 02 User](2-k8s-admin-user.md).

Welche Ressourcen namespaced sind, lässt sich abfragen:

```shell
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

## Namespace anlegen

```shell
kubectl create namespace my-namespace
```

Ressourcen lassen sich gezielt in einem Namespace anlegen oder abfragen:

```shell
kubectl get pods --namespace=my-namespace
kubectl apply -f pod.yaml --namespace=my-namespace
```

Alternativ direkt in der YAML-Beschreibung:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace
```

## Kontext wechseln

Ohne `--namespace` arbeitet `kubectl` immer im aktuell gesetzten Namespace des Kontexts, standardmäßig `default`. Damit nicht bei jedem Befehl der Namespace mit angegeben werden muss, lässt sich der Default-Namespace des Kontexts dauerhaft umstellen:

```shell
kubectl config set-context --current --namespace=my-namespace
```

Ab diesem Punkt beziehen sich alle `kubectl`-Befehle ohne explizite `--namespace`-Angabe auf `my-namespace`.

## ResourceQuota

Ein Namespace kann über eine `ResourceQuota` begrenzt werden, wie viele Ressourcen (Objekte oder Compute-Ressourcen) innerhalb von ihm insgesamt verbraucht werden dürfen. Das verhindert, dass ein einzelner Namespace den gesamten Cluster für andere Namespaces blockiert.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-namespace-quota
  namespace: my-namespace
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
```

Eine ResourceQuota wirkt nur innerhalb des Namespace, in dem sie definiert ist, und nur auf die dort erfassten Ressourcentypen.
