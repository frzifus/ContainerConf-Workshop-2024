# Workshop: Alles über OpenTelemetry

__Abstract__: Dieser Workshop behandelt Observability mit OpenTelemetry. Im Vordergrund steht dabei das Distributed Tracing, mit dem man Probleme in (stark) verteilten Anwendungen erkennen und behandeln kann.

Zu Beginn des Workshops werden die Grundlagen und zugrundeliegenden Konzepte erklärt. 
Im zweiten Teil betrachten wir uns die Instrumentierung von Software an Beispielen von Java/Quarkus und Python an. 
Der dritte Teil behandelt das Deployment in Kubernetes/OpenShift-Clustern sowie wie der Collector effizent eingesetzt werden kann.
Zuletzt widmen wir uns Themen wie Sampling und Filtering, um die Menge an anfallenden Daten in den Griff zu bekommen.

__Vorkenntnisse__: Generelles Verständnis von Verteilten Systemen und Programmierung. Spezielle Kenntnisse in Java oder Python sind nicht notwendig, aber hilfreich.

__Lernziele__: Verständnis über das Verstehen der konkreten Arbeit eines verteilten Systems. Dies beinhaltet die Fehlersuche, aber auch Feedback an die Entwickler für zukünftige Optimierungen.

__Schedule__: https://www.continuouslifecycle.de/lecture.php?id=22524

__Slides__: [pdf](slides.pdf)

__Recording__: N/A

## Agenda

Jeder Schritt des Tutorials befindet sich in einer eigenen Datei:


1. 09:00 Uhr: Registrierung und Begrüßungskaffee
1. [Einrichtung der Lernumgebung](00-prerequisites.md)
1. 10:00 Uhr: Beginn
1. [Überblick Observability & Einführung in Distributed Tracing]()
1. [Praktische Instrumentalisierung von (Quarkus)Java- und Python-Anwendungen]()
1. 12:30 - 13:30 Uhr: Mittagspause
1. [Datenerfassung mit OpenTelemetry in der Kubernetes/OpenShift Umgebung](30-otel-k8s.md)
1. 15:00 - 15:15 Uhr: Kaffeepause
1. [Sampling und Filtering: Optimierung der Observability in verteilten Systemen](40-sampling.md)
1. 16:15 - 16:30 Uhr: Kaffeepause
1. [Teilnehmer Projekt]()
1.  ca. 17:00 Uhr: Ende
