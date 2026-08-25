# Kubernetes advance: Ingress

|||
|---|---|
| Title | K8 Advance Ingress |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-advance-training-ingress](https://muellermh.wordpress.com/k8s-advance-training-ingress) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/) |
| Description | Diese Lektion vermittelt alle Informationen zum Thema Ingress |

## Ingress

Ein [Service](../k8s-basic/9-k8s-basic-service.md) vom Typ `NodePort` oder `LoadBalancer` veröffentlicht Pods bereits nach außen, allerdings jeweils einen ganzen Port pro Service. Ingress geht einen Schritt weiter und bündelt HTTP/HTTPS Routing für mehrere Services hinter einem einzigen Einstiegspunkt, anhand von Host und Pfad. Zusätzlich kann ein Ingress TLS Terminierung übernehmen.

## Voraussetzung: Ingress Controller

Ein Ingress Objekt allein bewirkt in Kubernetes nichts. Es beschreibt nur die gewünschten Routing Regeln. Damit diese Regeln tatsächlich wirksam werden, muss zusätzlich ein Ingress Controller im Cluster laufen (zum Beispiel nginx-ingress, Traefik oder ein cloud-spezifischer Controller). Der Controller beobachtet Ingress Objekte und konfiguriert daraufhin einen Proxy oder Loadbalancer entsprechend. Ohne laufenden Controller bleibt ein angelegtes Ingress Objekt wirkungslos.

## Ingress beschreiben

Ein Ingress ist wie ein Service ein REST Object und wird deklarativ per YAML erstellt.

### Host-basiertes Routing

Beim Host-basierten Routing wird anhand des angefragten Hostnamens entschieden, an welchen Service der Traffic geht:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-routing-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: shop.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: shop-service
            port:
              number: 80
  - host: blog.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: blog-service
            port:
              number: 80
```

Anfragen an `shop.example.com` werden hier an den Service `shop-service` geroutet, Anfragen an `blog.example.com` an `blog-service`.

### Pfad-basiertes Routing

Beim Pfad-basierten Routing entscheidet stattdessen der URL Pfad über das Ziel:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-routing-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

Anfragen unter `example.com/api` gehen an `api-service`, alle anderen Anfragen unter `example.com` an `frontend-service`. Host- und Pfad-basiertes Routing lassen sich in einem Ingress Objekt auch kombinieren.

`pathType` legt fest, wie der Pfad ausgewertet wird:

* `Prefix` (matched den Pfad als Präfix)
* `Exact` (matched nur exakt den angegebenen Pfad)
* `ImplementationSpecific` (Auswertung ist dem jeweiligen Ingress Controller überlassen)

Als Backend hinter jeder Regel steht immer ein [Service](../k8s-basic/9-k8s-basic-service.md), niemals direkt ein Pod. Der Ingress übernimmt also nur das Routing auf Ebene HTTP/HTTPS, das Weiterleiten an die konkreten Pods erledigt weiterhin der dahinterliegende Service.
