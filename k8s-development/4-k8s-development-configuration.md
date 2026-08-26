# Kubernetes development: Configuration

|||
|---|---|
| Title | K8 Development Configuration |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-development-training-configuration](https://muellermh.wordpress.com/k8s-development-training-configuration) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/tasks/configure-pod-container/security-context/](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) |
| Description | Diese Lektion vermittelt, wie ein Container per Security Context abgesichert wird, sowie den Entwicklerbezug zu ConfigMaps, Resources, Secrets und Service Accounts. |

## Config maps

Was eine ConfigMap ist und wie sie als Environment Variable oder Volume eingebunden wird, behandelt bereits [k8s-advance Kapitel 02 ConfigMap](../k8s-advance/2-k8s-advance-config-map.md). Dev-Tipp: Wird eine ConfigMap als Volume gemountet, zieht das Kubelet Änderungen automatisch nach, allerdings zeitverzögert. Wird dieselbe Konfiguration stattdessen als Environment Variable bezogen, bekommt der Container die Änderung erst nach einem Neustart des Pods mit — bei sich häufig ändernder Konfiguration ist ein gemountetes Volume deshalb oft die robustere Wahl.

## Resources

Was `requests` und `limits` bedeuten und wie sie sich bei Überschreitung auf CPU und Memory unterschiedlich auswirken, behandelt bereits [k8s-advance Kapitel 06 Resources](../k8s-advance/6-k8s-advance-resources.md). Dev-Tipp: Wer beim lokalen Testen kein `limits.memory` setzt, sieht ein OOMKill oft erst im Cluster — für die eigene Anwendung lohnt es sich, `limits.memory` schon während der Entwicklung realistisch zu setzen, statt es dem Betrieb als Überraschung zu überlassen.

## Security context

Ein `securityContext` schränkt ein, mit welchen Rechten ein Pod oder ein einzelner Container läuft. Er lässt sich auf zwei Ebenen setzen: auf Pod-Ebene gilt er als Default für alle enthaltenen Container, auf Container-Ebene überschreibt er diesen Default gezielt für einen einzelnen Container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: app
    image: my-app:1.0
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
```

Die wichtigsten Felder im Überblick:

* `runAsUser` — die numerische User-ID, mit der der Hauptprozess im Container läuft, statt der im Image definierten (häufig `root`, UID 0)
* `runAsNonRoot` — erzwingt, dass der Container nicht als `root` starten darf; startet das Image trotzdem als `root`, verweigert das Kubelet den Start des Containers
* `readOnlyRootFilesystem` — mountet das Root-Dateisystem des Containers read-only. Schreibzugriffe sind dann nur noch über explizit gemountete Volumes möglich, was die Angriffsfläche reduziert, falls ein Prozess im Container kompromittiert wird
* `allowPrivilegeEscalation` — unterbindet, dass ein Prozess im Container sich mehr Rechte verschaffen kann, als sein Elternprozess hatte (z. B. über SUID-Binaries)
* `capabilities` — steuert einzelne Linux-Capabilities feingranular statt pauschal `privileged` zu setzen. Übliches Muster ist `drop: [ALL]`, um dem Container zunächst alle Capabilities zu entziehen, und danach gezielt nur die tatsächlich benötigten wieder mit `add` hinzuzufügen (im Beispiel `NET_BIND_SERVICE`, um Ports unterhalb von 1024 ohne vollen Root binden zu dürfen)

Gesetzte Werte auf Container-Ebene haben Vorrang vor denen auf Pod-Ebene. Als Grundhaltung für die eigene Anwendung gilt: so wenig Rechte wie möglich, und jede Abweichung vom restriktiven Default (`root`, Schreibzugriff, zusätzliche Capabilities) muss fachlich begründet sein.

## Secrets

Was ein Secret ist und wie es als Volume, Environment Variable oder Image-Pull-Secret eingebunden wird, behandelt bereits [k8s-basic Kapitel 12 Secret](../k8s-basic/12-k8s-basic-secret.md). Wichtige Ergänzung dazu: Ein Secret ist standardmäßig **nur base64-kodiert, nicht verschlüsselt**. Base64 ist keine Verschlüsselung, sondern lediglich eine Textdarstellung — jeder mit Lesezugriff auf das Secret-Objekt (z. B. über `kubectl get secret -o yaml`) kann den Klartext ohne Passwort zurückgewinnen. Vertraulichkeit entsteht erst durch zusätzliche Maßnahmen wie Verschlüsselung des etcd-Datastore (Encryption at Rest) oder ein externes Secret-Management-System.

## Service accounts

Jeder Pod läuft unter einem `ServiceAccount`, über das er sich gegenüber dem API-Server authentisiert. Ist keiner angegeben, verwendet der Pod automatisch den `default`-ServiceAccount seines Namespace. Für eigene Anwendungen, die mit der Kubernetes-API sprechen (z. B. Controller, Operatoren), wird ein dedizierter ServiceAccount referenziert:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: serviceaccount-demo
spec:
  serviceAccountName: my-app-sa
  automountServiceAccountToken: false
  containers:
  - name: app
    image: my-app:1.0
```

Wie ein ServiceAccount selbst angelegt und über `Role`/`RoleBinding` mit Rechten versehen wird, behandelt bereits [k8s-admin Kapitel 02 User](../k8s-admin/2-k8s-admin-user.md).

Als Best Practice gilt `automountServiceAccountToken: false`, sobald die eigene Anwendung gar nicht mit der Kubernetes-API kommuniziert. Ohne diese Einstellung mountet Kubernetes das API-Token des ServiceAccounts automatisch in jeden Pod, auch wenn der Container es nie benutzt. Das Token liegt dann unnötig im Dateisystem des Containers und stellt ein zusätzliches Risiko dar, falls der Container kompromittiert wird — insbesondere, wenn der Pod (bewusst oder versehentlich) unter dem `default`-ServiceAccount läuft, dessen Rechte selten aktiv eingeschränkt werden.
