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
