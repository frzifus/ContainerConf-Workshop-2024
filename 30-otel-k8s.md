# Datenerfassung mit OpenTelemetry in der Kubernetes/OpenShift Umgebung

## Überblick über das OpenTelemetry-Projekt

### Architektur und Einsatz des OpenTelemetry Collectors

> Herstellerunabhängige Möglichkeit, Telemetriedaten zu empfangen, zu verarbeiten und zu exportieren.

![OpenTelemetry Collector](images/opentelemetry-collector.png)

Der OpenTelemetry Collector kann in einige Hauptkomponenten unterteilt werden.

- **Receivers**: Sammeln Daten von einer bestimmten Quelle, z. B. einer Anwendung oder Infrastruktur, und wandeln sie in [pData (Pipeline-Daten)](https://pkg.go.dev/go.opentelemetry.io/collector/consumer/pdata#section-documentation) um. Diese Komponente kann aktiv (z. B. Prometheus) oder passiv (OTLP) sein.
- **Processors**: Verarbeitet die von den Empfängern gesammelten Daten in irgendeiner Weise. Ein Prozessor kann zum Beispiel irrelevante Daten herausfiltern oder Metadaten hinzufügen, um die Analyse zu erleichtern. Wie der Batch- oder Metrik-Umbenennungsprozessor.
- **Exporters**: Senden Daten zur Speicherung oder Analyse an ein externes System. Beispiele sind Prometheus, Loki oder der OTLP-Exporter.
- **Extensions**: Fügen Sie zusätzliche Funktionen zu OpenTelemetry hinzu, wie z. B. die Konfiguration eines Bearer-Tokens oder das Anbieten eines Jaeger-Endpunkts für die Fernabfrage.
- **Connectors**: Ist sowohl ein Exporteur als auch ein Empfänger. Er konsumiert Daten als Exporteur in einer Pipeline und sendet Daten als Empfänger in einer anderen Pipeline.

Weitere Einzelheiten: [offiziellen Dokumentation](https://opentelemetry.io/docs/collector/).

Die verfügbaren Komponenten werden in [Distributionen](https://opentelemetry.io/docs/concepts/distributions/) zusammengefasst. Das OpenTelemetry Projekt stellt derzeit 3 verschiedene Distributionen zur Verfügung.

- Core
- Contrib
- Kubernetes

Mit Hilfe des [OpenTelemetry Collector Builders](https://github.com/open-telemetry/opentelemetry-collector/tree/v0.103.0/cmd/builder) können eigene Distributionen erstellt werden.

### Konfiguration und Deployment des Collectors

Die Konfiguration des OpenTelemetry Collectors wird in YAML beschrieben. Das folgende Beispiel zeigt einen OTLP/gRPC-Receiver, der auf `localhost:4317` lauscht, einen Batch-Prozessor mit den Standardparametern und einen Debug-Exporter mit normalem Log-Level. Zudem werden mehrere Pipelines für verschiedene Telemetriedaten beschrieben, die ihre gesammelten Telemetriedaten alle an den Debug-Exporter weiterleiten.

Um mehr über die Konfigurationsoptionen der einzelnen Komponenten zu erfahren, empfiehlt es sich, das README im jeweiligen Komponentenordner direkt anzusehen, z.B. für den [debugexporter](https://github.com/open-telemetry/opentelemetry-collector/tree/v0.103.0/exporter/debugexporter).

```yaml
---
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
processors:
  batch:

exporters:
  debug:
    verbosity: normal

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
```

#### Collector Lokal ausführen

Hier starten wir einen Collector, der über `localhost` erreichbar ist, mit der oben gezeigten Konfiguration:

```bash
podman run --rm -it --name otel-collector -p 4317:4317 -p 4318:4318 ghcr.io/open-telemetry/opentelemetry-collector-releases/opentelemetry-collector:0.103.0 --config https://raw.githubusercontent.com/frzifus/ContainerConf-Workshop-2024/main/collector-config.yaml
```

#### Telemetriedaten an den Collector senden

Der zuvor konfigurierte Collector lauscht nun auf `localhost:4317` ohne TLS. Um zu testen, ob der Collector tatsächlich Metriken, Logs und Traces empfängt und an den angegebenen Logging-Exporter weiterleitet, können wir mit telemetrygen Testdaten generieren.

```bash
telemetrygen metrics --otlp-insecure --duration 10s --rate 4
# or
telemetrygen logs --otlp-insecure --duration 10s --rate 4
# or
telemetrygen traces --otlp-insecure --duration 10s --rate 4
```

Falls `telemetrygen` nicht installiert ist, kann alternativ auch das Container-Image verwendet werden:

```bash
docker run --rm -it --link otel-collector ghcr.io/open-telemetry/opentelemetry-collector-contrib/telemetrygen:v0.103.0 metrics --otlp-endpoint=otel-collector:4317 --otlp-insecure --duration 10s --rate 4
# or
docker run --rm -it --link otel-collector ghcr.io/open-telemetry/opentelemetry-collector-contrib/telemetrygen:v0.103.0 logs --otlp-endpoint=otel-collector:4317 --otlp-insecure --duration 10s --rate 4
# or
docker run --rm -it --link otel-collector ghcr.io/open-telemetry/opentelemetry-collector-contrib/telemetrygen:v0.103.0 traces --otlp-endpoint=otel-collector:4317 --otlp-insecure --duration 10s --rate 4
```

Erwartete Ausgabe:

```
2024-11-11T12:27:26.638Z	info	LogsExporter	{"kind": "exporter", "data_type": "logs", "name": "logging", "#logs": 1}
2024-11-11T12:27:30.248Z	info	LogsExporter	{"kind": "exporter", "data_type": "logs", "name": "logging", "#logs": 2}
2024-11-11T12:27:34.457Z	info	MetricsExporter	{"kind": "exporter", "data_type": "metrics", "name": "logging", "#metrics": 2}
2024-11-11T12:27:34.857Z	info	MetricsExporter	{"kind": "exporter", "data_type": "metrics", "name": "logging", "#metrics": 1}
2024-11-11T12:27:39.468Z	info	TracesExporter	{"kind": "exporter", "data_type": "traces", "name": "logging", "#spans": 8}
2024-11-11T12:27:41.473Z	info	TracesExporter	{"kind": "exporter", "data_type": "traces", "name": "logging", "#spans": 10}
```

## Datenerfassung in Kubernetes und OpenShift

```bash
$ kubectl get nodes
NAME                     STATUS   ROLES           AGE     VERSION
workshop-control-plane   Ready    control-plane   3m23s   v1.29.1
```


### Einführung des Operators: Konfiguration und Deployment

```bash
kubectl apply -f https://raw.githubusercontent.com/frzifus/ContainerConf-Workshop-2024/main/app/install.yaml
```

```
kubectl port-forward svc/replicator 8080:8080
kubectl port-forward -n observability-backend svc/jaeger-query 16686:16686
```

TODO: Spanmetricsconnector
```bash

```

### Integration mit Service- und PodMonitors
