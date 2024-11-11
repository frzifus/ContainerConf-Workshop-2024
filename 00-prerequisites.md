# Voraussetzungen

Um diesen Workshop erfolgreich durchführen zu können, sollten die Teilnehmer die folgenden Voraussetzungen erfüllen:

1. Eine stabile Internetverbindung ist erforderlich, da während des Workshops etwa 5 GB an Daten heruntergeladen werden.
1. Die Möglichkeit, Container lokal auszuführen, entweder mit [Docker](https://docs.docker.com/engine/install/) oder [Podman](https://podman.io/docs/installation).
1. Zugang zu einem Kubernetes- oder OpenShift-Cluster. Falls kein Zugang vorhanden ist, können lokale Testcluster mithilfe von [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) und [Minikube](https://minikube.sigs.k8s.io/docs/start/) erstellt werden. OpenShift-Cluster lassen sich lokal mit [CRC](https://crc.dev/crc/getting_started/getting_started/installing/) bereitstellen.
1. Eine aktuelle Version des Befehlszeilentools [kubectl](https://kubernetes.io/docs/tasks/tools/) oder [oc](https://docs.openshift.com/container-platform/4.16/cli_reference/openshift_cli/getting-started-cli.html) sollte installiert sein.
1. PostgeSQL in container ( quay.io/fedora/postgresql-16 )
2. Mindestens eines von
   * OpenJDK 17+ / Maven 3.8+ / Quarkus 3.15 / Java Editor oder IDE
   * Python 3.8+ / Python Editor oder IDE

# Kubernetes Setup mit Kind

### Kind Installation

Falls Go auf deinem Rechner installiert ist, kann Kind ganz einfach wie folgt installiert werden:

```bash
go install sigs.k8s.io/kind@v0.29.0
```

Falls Go nicht installiert ist, lade die [kind-v0.29.0](https://github.com/kubernetes-sigs/kind/releases/tag/v0.29.0) Binärdatei von der Release-Seite herunter. (Andere Versionen werden wahrscheinlich ebenfalls funktionieren. 🤠)

## Workshop-Cluster erstellen

Nach einer erfolgreichen Installation kann ein Cluster wie folgt erstellt werden:

```bash
kind create cluster --name=workshop --config=kind-1.29.yaml
```

Kind setzt automatisch den Kube-Kontext auf den neu erstellten Workshop-Cluster. Wir können dies einfach überprüfen, indem wir Informationen über unsere Nodes abrufen:

```bash
kubectl get nodes
```

Erwartete Ausgabe:
```bash
NAME                     STATUS   ROLES           AGE   VERSION
workshop-control-plane   Ready    control-plane   75s   v1.29.1
```

## Aufräumen

Um den Cluster wieder zu löschen:

```bash
kind delete cluster --name=workshop
```

# Bereitstellung der initialen Services

## Deployment von cert-manager

Die `cert-manager` installation wird vom OpenTelemetry-Operator benötigt, um TLS-Zertifikate für Admission-Webhooks zu erhalten.

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.1/cert-manager.yaml
kubectl get pods -n cert-manager -w 
```

## Deployment des OpenTelemetry Operators

```bash
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/download/v0.103.0/opentelemetry-operator.yaml
kubectl get pods -n opentelemetry-operator-system -w  
```

## Deployment des Observability-Backends

Die Installation des Observability-Backends ist optional. Ein OTLP-kompatibles Backend für Trace- und Metrikdaten wird jedoch vorausgesetzt. 
Falls keines vorhanden ist, können Prometheus für Metriken und Jaeger für Traces wie folgt installiert werden:

```bash
kubectl apply -f https://raw.githubusercontent.com/frzifus/ContainerConf-Workshop-2024/main/backend/01-backend.yaml
kubectl get pods -n observability-backend -w 
```

Nach der Installation ist das Backend im Namespace observability-backend zu finden.

Um das Jaeger-UI im Browser aufzurufen, kann folgender Befehl zum Port-Forwarding verwendet werden:

```bash
kubectl port-forward -n observability-backend service/jaeger-query 16686:16686
```

Jaeger ist dann lokal unter [localhost:16686](http://localhost:16686) erreichbar.
