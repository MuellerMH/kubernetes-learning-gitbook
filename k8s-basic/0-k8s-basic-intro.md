# Kubernetes Basic Training Intro

| Duration | 2:00 Minutes |
| YouTube | [Basic Training Intro](https://www.youtube.com/watch?v=ia3VFuQpaSQ) |
| Blog | https://muellermh.wordpress.com/k8s-basic-training-node  |
| Author | Manuel H. Müller alias Onko |
| Mail | mm@kubernauts.de |
| Description | By the end of this course you will understand the basic topics of k8s. You will learn how container, pods, replica, services, secrets, labels and deployments works also with helath-checks.   |

## What awaits you in this course

- Kubernetes Master Basics
- Kubernetes Etcd Key Value Storage
- Kubernetes Nodes
- Docker and Container
- Kubernetes Pods
- Kubernetes Replica Controller
- Kubernetes Replica Sets
- Kubernetes Deployment
- Kubernetes Service
- Kubernetes Annotation
- Kubernetes Label
- Kubernetes Secret
- Kubernetes Job
- Kubernetes DeamonSet
- Kubernetes Bare Pod
- Kubernetes Health Check
- Kubernetes Volumes

## Important to save time

generate a secret yaml

```bash
kubectl create secret generic test-secret --from-literal=foo=bar -o yaml --dry-run=client > test-secret.yaml
```

generate a deployment yaml

```bash
kubectl create deployment test-deployment --image=nginx -o yaml --dry-run=client > test-deployment.yaml
```

generate a service nodeport yaml

```bash
kubectl create service nodeport test-service --tcp 80:8080 -o yaml --dry-run=client > test-service.yaml
```

generate a pod yaml

```bash
kubectl run test-pod --image=nginx -o yaml --dry-run=client > test-pod.yaml
```

generate a job yaml

```bash
kubectl create job test-job --image=busybox -o yaml --dry-run=client -- echo hello > test-job.yaml
```

## Voraussetzungen

Für die praktischen Übungen brauchst du einen Test-Cluster und `kubectl`. Für den Einstieg reicht ein lokaler Cluster:

- [kind](https://kind.sigs.k8s.io/) — Kubernetes in Docker, `kind create cluster`
- [minikube](https://minikube.sigs.k8s.io/) — `minikube start`
- [k3d](https://k3d.io/) — k3s in Docker, `k3d cluster create`

Mit `kubectl cluster-info` und `kubectl get nodes` prüfst du, ob die Verbindung zum Cluster steht.

## Jetzt abonnieren

und das nächste Video nicht verpassen!
