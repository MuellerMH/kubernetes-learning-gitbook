# Kubernetes admin: User

|||
|---|---|
| Title | K8 Admin User |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-admin-training-user](https://muellermh.wordpress.com/k8s-admin-training-user) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Role Based Access Control (RBAC) |

## Role Based Access Control (RBAC)

Wie im [Namespace-Kapitel](1-k8s-admin-namespace.md) bereits angesprochen, ist ein Namespace für sich genommen keine Security-Boundary. Wer festlegen will, welcher Nutzer oder welcher Prozess innerhalb eines Namespace oder clusterweit was tun darf, braucht **RBAC** (Role Based Access Control). RBAC entscheidet auf Basis von vier Objekttypen, ob eine Anfrage an den API-Server erlaubt ist: `Role`, `ClusterRole`, `RoleBinding` und `ClusterRoleBinding`.

Eine Rolle beschreibt dabei nur, welche Aktionen (`verbs`) auf welchen Ressourcen erlaubt sind. Wer diese Rolle tatsächlich bekommt, legt erst das dazugehörige Binding fest — Rolle und Bindung sind bewusst getrennte Objekte.

## Role vs. ClusterRole

Eine `Role` ist wie im [Namespace-Kapitel](1-k8s-admin-namespace.md) erwähnt **namespaced**: Sie gilt nur innerhalb des Namespace, in dem sie definiert wurde.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: my-namespace
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
```

Eine `ClusterRole` ist dagegen **cluster-scoped** und existiert nur einmal für den gesamten Cluster — genau wie `Node` oder `PersistentVolume`. Sie eignet sich für zwei Fälle: Rechte, die sich auf cluster-scoped Ressourcen beziehen (z. B. `Node`), oder Rechte, die in mehreren Namespaces gleichermaßen gelten sollen.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-only-viewer
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "configmaps", "services"]
  verbs: ["get", "list", "watch"]
```

Welche `apiGroups` und `resources` es gibt, lässt sich abfragen:

```shell
kubectl api-resources
```

## ServiceAccount

Ein `User` in Kubernetes ist kein eigenes API-Objekt — er wird extern verwaltet (Zertifikat, OIDC-Provider, Cloud-IAM) und existiert für den Cluster nur als Name in einem Request. Für Prozesse, die innerhalb des Clusters laufen und selbst mit dem API-Server sprechen (Pods, Controller, CI/CD-Jobs), gibt es dagegen ein echtes, namespaced API-Objekt: das `ServiceAccount`.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pipeline-deployer
  namespace: my-namespace
```

Jeder Pod läuft unter einem ServiceAccount — wird keins angegeben, verwendet er das `default`-ServiceAccount des Namespace. Das Token des ServiceAccount wird dem Pod automatisch eingebunden und für Zugriffe auf den API-Server genutzt.

```shell
kubectl get serviceaccounts --namespace=my-namespace
```

## RoleBinding vs. ClusterRoleBinding

Ein `RoleBinding` verknüpft eine `Role` (oder auch eine `ClusterRole`, deren Wirkung dadurch auf den Namespace des Bindings beschränkt bleibt) mit einem oder mehreren Subjekten — `User`, `Group` oder `ServiceAccount`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pipeline-deployer-binding
  namespace: my-namespace
subjects:
- kind: ServiceAccount
  name: pipeline-deployer
  namespace: my-namespace
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: rbac.authorization.k8s.io
```

Ein `ClusterRoleBinding` verknüpft dagegen eine `ClusterRole` clusterweit mit einem Subjekt — die Rechte gelten dann in allen Namespaces gleichzeitig.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-read-only-binding
subjects:
- kind: ServiceAccount
  name: monitoring-agent
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: read-only-viewer
  apiGroup: rbac.authorization.k8s.io
```

> Früher wurde RBAC in vielen Tutorials am Beispiel von Tiller (Helm 2) demonstriert. Tiller wurde mit Helm 3 im Jahr 2019 vollständig entfernt und existiert in aktuellen Helm-Versionen nicht mehr — das folgende Beispiel nutzt deshalb ein Szenario, das es in heutigen Clustern tatsächlich gibt.

## Praxisbeispiel: CI/CD-Pipeline mit eingeschränktem ServiceAccount

Eine CI/CD-Pipeline (z. B. ein Runner, der nach jedem Merge ein Deployment aktualisiert) soll in einem Namespace Deployments und ConfigMaps verwalten dürfen, aber sonst keine weiteren Rechte im Cluster haben.

#### Namespace anlegen

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

#### ServiceAccount für die Pipeline anlegen

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pipeline-deployer
  namespace: my-namespace
```

#### Role mit den benötigten Rechten anlegen

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: my-namespace
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
```

#### RoleBinding zwischen ServiceAccount und Role anlegen

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pipeline-deployer-binding
  namespace: my-namespace
subjects:
- kind: ServiceAccount
  name: pipeline-deployer
  namespace: my-namespace
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: rbac.authorization.k8s.io
```

Damit kann der Pipeline-ServiceAccount ausschließlich Deployments und ConfigMaps in `my-namespace` verwalten. Auf Pods, Secrets oder Ressourcen in anderen Namespaces hat er keinen Zugriff, weil weder die Role noch das Binding dafür Rechte vergeben.

#### Rechte überprüfen

Ob eine Aktion tatsächlich erlaubt ist, lässt sich testen, ohne die Aktion selbst auszuführen:

```shell
kubectl auth can-i update deployments \
  --namespace=my-namespace \
  --as=system:serviceaccount:my-namespace:pipeline-deployer

kubectl auth can-i delete secrets \
  --namespace=my-namespace \
  --as=system:serviceaccount:my-namespace:pipeline-deployer
```

Der erste Befehl liefert `yes`, der zweite `no` — genau das Verhalten, das die Role oben definiert.
