# Kubernetes Basic Training Intro

| Duration | 2:00 Minutes |
| YouTube | [Basic Training Intro](https://www.youtube.com/watch?v=ia3VFuQpaSQ) |
| Blog | https://muellermh.wordpress.com/k8s-basic-training-node  |
| Author | Manuel H. Müller alias Onko |
| Mail | mm@kubernauts.de |
| Description | By the end of this course you will understand the basic topics of k8s. You will learn how container, pods, replica, services, secrets, labels and deployments works also with helath-checks.   |

## Importend to save time

generate a secret yaml

```bash
kubectl create secret generic test-secret --from-literal=foo -o yaml --dry-run > test-secret.yaml
```

generate a deployment yaml

```bash
kubectl create deployment test-deployment --image=nginx -o yaml --dry-run > test-deployment.yaml
```

generate a service nodeport yaml

```bash
kubectl create service nodeport test-service --tcp 80:8080 -o yaml --dry-run > test-service.yaml
```
=======
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
