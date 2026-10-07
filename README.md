# Learning Kubernetes

This is a free collection of notes and exercises for preparing the CKA / CKAD certifications, provided by [CodeCraft Academy](https://codecraft.academy) as an open resource for the Kubernetes learning community.

[Book Summary](SUMMARY.md)

See the rendered version on the [Kubernetes Learning GitBook Pages](https://muellermh.github.io/kubernetes-learning-gitbook/).

## Structure of the book

| Section | Target | Topics |
|---|---|---|
| [Basics](k8s-basic/0-k8s-basic-intro.md) | Beginners | Architecture, Pods, Controllers, Services, Labels, Secrets, Volumes |
| [Advance](k8s-advance/0-k8s-advance-intro.md) | Advanced users | Discovery, ConfigMaps, Ingress, Storage, DaemonSets, Resources, Autoscaling |
| [Admin](k8s-admin/0-k8s-admin-intro.md) | CKA | Namespaces, RBAC, Networking, High Availability, Nodes, Cluster Maintenance |
| [Development](k8s-development/0-k8s-development-intro.md) | CKAD | Pod design, Multi-container pods, Configuration, Observability, Services, Networking |

## Quick overview kubectl

| Command | Purpose | Example |
|---|---|---|
| `get` | list resources | `kubectl get pods,deployments,rs,svc -o wide` |
| `describe` | details and events of a resource | `kubectl describe pod nginx` |
| `create` | create a resource imperatively | `kubectl create deployment nginx --image=nginx` |
| `apply` | create or update from a file | `kubectl apply -f deployment.yaml` |
| `expose` | create a service for a resource | `kubectl expose deployment nginx --port=80 --type=NodePort` |
| `scale` | change the number of replicas | `kubectl scale deployment nginx --replicas=3` |
| `set image` | update the image of a container | `kubectl set image deployment/nginx nginx=nginx:1.27` |
| `rollout` | `status`, `history`, `undo` of a rollout | `kubectl rollout undo deployment/nginx` |
| `label` | add or change labels | `kubectl label pod nginx env=prod` |
| `logs` | show container logs | `kubectl logs -f nginx` |
| `exec` | run a command in a container | `kubectl exec -it nginx -- /bin/bash` |
| `delete` | remove resources | `kubectl delete -f deployment.yaml` |
| `explain` | documentation of a field | `kubectl explain pod.spec.containers` |

Generate YAML instead of writing it by hand:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

## Other tools

### curl

Inside the cluster a temporary pod with `curl` is the quickest way to test a service:

```bash
kubectl run tmp --rm -it --restart=Never --image=curlimages/curl -- curl -s http://nginx.default.svc.cluster.local
```

From your machine you can reach a service or pod through a port-forward:

```bash
kubectl port-forward svc/nginx 8080:80
curl http://localhost:8080
```

## Workflow

1. Create a deployment: `kubectl create deployment nginx --image=nginx --replicas=2`
2. Expose it as a service: `kubectl expose deployment nginx --port=80`
3. Check that the service has endpoints: `kubectl get endpoints nginx`
4. Test it with `curl` (see above)
5. Roll out a new version: `kubectl set image deployment/nginx nginx=nginx:1.27` and `kubectl rollout status deployment/nginx`

## License & Attribution

This material is provided by CodeCraft Academy, part of MHM Digitale Lösungen UG, for free use by the Kubernetes community. It is licensed under the MIT License — see [LICENSE](LICENSE).
