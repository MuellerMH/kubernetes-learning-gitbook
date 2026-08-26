# Kubernetes development: Core concept

|||
|---|---|
| Title | K8 Development Core Concept |
| Category | Course |
| Level | proficient |
| Duration | ? |
| YouTube | [https://youtube.com/crankzone/xxx](https://youtube.com/crankzone/xxx) |
| Blog | [https://muellermh.wordpress.com/k8s-development-training-coreconcept](https://muellermh.wordpress.com/k8s-development-training-coreconcept) |
| Author | Manuel H. "Onko" Müller |
| Mail | mm@kubernauts.de |
| Resource | [https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) |
| Description | Diese Lektion vermittelt die Entwicklerperspektive auf den Pod: Lifecycle, Container-States und die Konfiguration von Command, Args und Environment. |

## Pod overview

Was ein Pod ist und wie er sich zu Container/Volumes verhält, behandelt bereits [k8s-basic Kapitel 05 Pod](../k8s-basic/5-k8s-basic-pod.md). Aus Entwicklersicht stellt sich vor allem eine Frage: wie viele Container gehören in denselben Pod?

Ein Pod bildet die Deployment-Einheit für eine zusammengehörige Gruppe von Prozessen ab, nicht für eine ganze Anwendung. Die Faustregel: Ein Container reicht, solange die App als ein Prozess skaliert und deployed werden kann. Ein zweiter Container gehört nur dann in denselben Pod, wenn er eine echte Laufzeit-Abhängigkeit zum Hauptprozess hat, zum Beispiel einen gemeinsamen Datenaustausch über `localhost` oder ein gemeinsames Volume erfordert und immer zusammen mit dem Hauptcontainer skaliert und deployed werden soll. Frontend und Backend einer Anwendung skalieren in der Regel unabhängig voneinander und gehören deshalb in getrennte Pods mit eigenen Deployments. Mehr zu den Mustern, wann ein zweiter Container sinnvoll ist, behandelt [Kapitel 02 Multi Container Pod](2-k8s-development-multicontainerpod.md).

## Pod lifecycle

Jeder Pod durchläuft während seines Bestehens eine von fünf Phasen:

Phase | Beschreibung
--- | ---
Pending | Der Pod wurde vom API-Server akzeptiert, mindestens ein Container ist aber noch nicht erstellt (Scheduling, Image-Pull)
Running | Der Pod wurde auf einen Node gebunden, alle Container sind erstellt und mindestens einer läuft, startet gerade oder wird neu gestartet
Succeeded | Alle Container im Pod haben sich erfolgreich beendet und werden nicht neu gestartet
Failed | Alle Container im Pod haben sich beendet, mindestens einer davon mit einem Fehlerstatus
Unknown | Der Status des Pods konnte nicht ermittelt werden, meist wegen eines Kommunikationsfehlers mit dem Node

Zusätzlich zur Pod-Phase hat jeder einzelne Container einen eigenen State:

State | Beschreibung
--- | ---
Waiting | Der Container ist noch nicht lauffähig, zum Beispiel weil das Image noch gezogen wird oder ein Secret fehlt
Running | Der Container läuft ohne Probleme
Terminated | Der Container wurde beendet, entweder erfolgreich oder mit einem Fehler

Ob und wie ein beendeter Container neu gestartet wird, steuert `restartPolicy` auf Pod-Ebene:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restart-demo
spec:
  restartPolicy: OnFailure
  containers:
  - name: app
    image: nginx
```

Wert | Beschreibung
--- | ---
Always | Container wird nach jedem Ende neu gestartet (Default, üblich bei Deployments)
OnFailure | Container wird nur bei einem Fehler-Exit neu gestartet, nicht bei sauberem Beenden (üblich bei Jobs)
Never | Container wird nach seinem Ende nicht neu gestartet

### Praxisbezug: CrashLoopBackOff

Startet ein Container mit `restartPolicy: Always` wiederholt und beendet sich dabei jedes Mal mit einem Fehler, zeigt `kubectl get pods` den Status `CrashLoopBackOff`. Kubernetes wartet dabei zwischen den Neustartversuchen eine exponentiell wachsende Zeitspanne, um den Node nicht mit Neustartversuchen zu überlasten. Für die Fehlersuche sind zwei Befehle der erste Schritt:

```shell
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
```

`describe` zeigt die Events (z. B. `Back-off restarting failed container`) und den letzten `Terminated`-State inklusive Exit-Code und Reason, `logs --previous` zeigt die Ausgabe des vorherigen, bereits abgestürzten Containers, da der aktuelle Log-Stream nach einem Neustart wieder bei null beginnt. Mehr zur Log- und Debugging-Praxis behandelt [Kapitel 06 Observability](5-k8s-development-observability.md).

## Pod and container config

### command und args

Ein Container-Image bringt in der Regel bereits einen Standard-Startbefehl mit, definiert über `ENTRYPOINT` und `CMD` im Dockerfile. In der Pod-Spezifikation lässt sich dieser Standard gezielt überschreiben:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c"]
    args: ["echo hello && sleep 3600"]
```

`command` überschreibt dabei den `ENTRYPOINT` des Images, `args` überschreibt `CMD`. Wird nur `args` gesetzt, bleibt der `ENTRYPOINT` des Images erhalten und die `args` werden ihm als Parameter übergeben. Wird nur `command` gesetzt, wird ausschließlich dieser Befehl ausgeführt, ein etwaiges `CMD` aus dem Image wird ignoriert.

### Environment Variablen

Environment Variablen lassen sich direkt im Pod setzen:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: APP_MODE
      value: production
```

Statt eines festen Werts kann eine Env-Variable auch aus einer ConfigMap, einem Secret oder aus Feldern des Pods selbst befüllt werden (`valueFrom`). Wie ConfigMaps und Secrets als Env-Variable eingebunden werden, behandelt [Kapitel 05 Configuration](4-k8s-development-configuration.md).

### Resources und Security Context

Zwei weitere Bausteine der Container-Konfiguration werden an dieser Stelle nur benannt, da sie einen eigenen Abschnitt verdienen: die Ressourcen-Steuerung über `resources.requests`/`resources.limits` und die Absicherung des Containers über `securityContext`. Beides behandelt ausführlich [Kapitel 05 Configuration](4-k8s-development-configuration.md).
