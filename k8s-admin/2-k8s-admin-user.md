# Role Based Access Controll ( RBAC )

---

## Roles

---

## Service Accounts

---

## Example Tiller

---

### Setup Role

--

#### Create Name Space

--

```yaml
kind: Namespace
apiVersion: v1
metadata:
    name: mytiller
```

--

#### Create Service Account

--

```yaml
kind: ServiceAccount
apiVersion: v1
metadata:
    name: tiller-sa
```

--

#### Create Roles

--

Allow to manage all resources

--

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tiller-manager
  namespace: mytiller
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
```

--

Access to read configmaps

--

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: mytiller
  name: tiller-configmaps
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["configmaps"]
  verbs: ["*"]
```

--

#### Create RoleBinding

--

Bind allow to manage resources

--

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tiller-manager
  namespace: mytiller
subjects:
- kind: ServiceAccount
  name: tiller-sa
  namespace: mytiller
roleRef:
  kind: Role
  name: tiller-manager
  apiGroup: rbac.authorization.k8s.io

```

--

Bind access to configmaps

--

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tiller-binding
  namespace: mytiller
subjects:
- kind: ServiceAccount
  name: tiller-sa
  namespace: mytiller
roleRef:
  kind: Role
  name: tiller-configmaps
  apiGroup: rbac.authorization.k8s.io
```


## Helm cmds

Init Helm for tls

```shell
helm init --tiller-tls --tiller-tls-cert ./tiller.cert.pem --tiller-tls-key ./tiller.key.pem --tiller-tls-verify --tls-ca-cert ca.cert.pem --tiller-namespace=mytiller --service-account=tiller-sa --client-only
```

try a ls call

```shell
helm ls --tls --tls-ca-cert ca.cert.pem --tls-cert helm.cert.pem --tls-key helm.key.pem
```
