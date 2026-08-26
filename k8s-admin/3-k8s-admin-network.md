# Kubernetes admin: Network

|||
|---|---|
| Title | K8 Admin Network |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-admin-training-network](https://muellermh.wordpress.com/k8s-admin-training-network) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Netzwerk und NetworkPolicy |

## Grundmodell

Kubernetes verlangt von jeder Netzwerk-Implementierung ein einfaches Grundmodell:

* Jeder Pod bekommt eine eigene, clusterweit eindeutige IP-Adresse
* Alle Pods können alle anderen Pods im Cluster ohne NAT erreichen, unabhängig davon, auf welchem Node sie laufen
* Ein Node kann jeden Pod im Cluster ohne NAT erreichen
* Die IP, die ein Pod sich selbst sieht, ist dieselbe, unter der ihn andere erreichen

Das Ergebnis ist ein flaches Netz: Pods verhalten sich netzwerktechnisch wie eigenständige Hosts, ohne dass sich Anwendungen um Port-Mapping oder NAT-Übersetzung kümmern müssen.

## CNI

Kubernetes selbst implementiert dieses Netzwerkmodell nicht, sondern delegiert es an ein **CNI-Plugin** (Container Network Interface). Das CNI-Plugin richtet beim Start eines Pods dessen Netzwerk-Namespace ein, vergibt die IP und sorgt für die Erreichbarkeit im Cluster.

Es existieren mehrere CNI-Implementierungen mit unterschiedlichem Funktionsumfang, unter anderem Calico, Flannel und Weave. Sie unterscheiden sich unter anderem darin, ob sie NetworkPolicy unterstützen, welches Overlay- oder Routing-Verfahren sie nutzen und welche zusätzlichen Features (z. B. Verschlüsselung) sie mitbringen. Welches Plugin für einen konkreten Cluster passt, hängt von der jeweiligen Umgebung und den Anforderungen ab.

## kube-proxy

`kube-proxy` läuft als Agent auf jedem Node und setzt die Service-Abstraktion netzwerktechnisch um: Es sorgt dafür, dass Traffic, der an eine Service-IP geht, an einen der passenden Pods weitergeleitet wird. Dafür pflegt `kube-proxy` je nach Modus Regeln über `iptables` oder `IPVS` im Kernel des Nodes. Wie Services selbst funktionieren, behandelt bereits [k8s-basic Kapitel 09 Service](../k8s-basic/9-k8s-basic-service.md).

## CoreDNS

`CoreDNS` läuft als Deployment im Namespace `kube-system` und stellt die clusterinterne DNS-Auflösung bereit. Services und teils auch Pods bekommen automatisch einen DNS-Namen, über den sie sich innerhalb des Clusters ansprechen lassen, ohne die konkrete IP kennen zu müssen.

```shell
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

## NetworkPolicy

Standardmäßig darf im flachen Netzwerkmodell jeder Pod mit jedem anderen Pod kommunizieren. Eine `NetworkPolicy` schränkt das gezielt ein: Sie definiert Regeln, welcher Traffic zu (`Ingress`) und von (`Egress`) einer über `podSelector` ausgewählten Menge von Pods erlaubt ist.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-only
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

Diese NetworkPolicy erlaubt eingehenden Traffic auf Port `8080` zu Pods mit dem Label `app: backend` nur von Pods mit dem Label `app: frontend`. Alle anderen Quellen sind für diesen Traffic gesperrt, sobald mindestens eine NetworkPolicy mit `Ingress` in `policyTypes` auf einen Pod zutrifft.

> Wichtig: Eine NetworkPolicy wirkt nur, wenn das eingesetzte CNI-Plugin sie auch tatsächlich umsetzt. Nicht jedes CNI-Plugin unterstützt NetworkPolicy — legt man eine NetworkPolicy auf einem Cluster mit einem Plugin ohne diese Unterstützung an, wird sie von Kubernetes zwar als Objekt gespeichert, aber vom Netzwerk-Stack schlicht ignoriert. Vor dem Einsatz von NetworkPolicy lohnt sich deshalb ein Blick in die Dokumentation des eingesetzten CNI-Plugins.
