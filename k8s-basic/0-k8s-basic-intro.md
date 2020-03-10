# Kubernetes Basic Training Intro

|||
|---|---|
| Title | K8 Basic Training |
| Category | Course |
| Level | Novice |
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