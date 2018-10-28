# Kubernetes Basic Training Intro

---

|||
|---|---|
| Title | K8 Basic Training |
| Category | Course |
| Level | Novice |
| Description | By the end of this course you will understand the basic topics of k8s.|

---

## What awaits you in this course

--

### Kubernetes Master Basics

--

### Kubernetes Etcd Key Value Storage

--

### Kubernetes Nodes

--

### Docker and Container

--

### Kubernetes Pods

--

### Kubernetes Replica Controller

--

### Kubernetes Replica Sets

--

### Kubernetes Deployment

--

### Kubernetes Service

--

### Kubernetes Annotation

--

### Kubernetes Label

--

### Kubernetes Secret

--

### Kubernetes Job

--

### Kubernetes DeamonSet

--

### Kubernetes Bare Pod

--

### Kubernetes Health Check

---

## Importend to save time

--

generate a secret yaml

--

```bash
kubectl create secret generic mysecret
--from-literal=foo
-o yaml --dry-run > secret.yaml
```

--

generate a deployment yaml

--

```bash
kubectl create deployment mydeployment
--image=nginx
-o yaml --dry-run > deployment.yaml
```

--

generate a service nodeport yaml

--

```bash
kubectl create service nodeport myservice
--tcp 80:8080
-o yaml --dry-run > service.yaml
```

---

## Jetzt abonnieren

und das nächste Video nicht verpassen!