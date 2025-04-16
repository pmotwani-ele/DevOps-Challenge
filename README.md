# 🛠️ Kafka DevOps Setup – Infrastructure Documentation

This repository sets up a Kafka-based message-driven architecture using:

- Apache Kafka cluster via Helm
- Producer and Consumer Python applications
- Kubernetes manifests and Helm-based IaC
- Local development stack with Docker Compose

---

## ☁️ Kafka Cluster Setup (Helm)

### ✅ Kafka Version

Kafka is installed via the official [Strimzi Helm Chart](https://strimzi.io/).  
Version is managed through the `values.yaml` file:

kafka:
  version: 3.8.1

###  To upgrade Kafka

  Update the version field in values.yaml.

  Re-run the Helm upgrade command:

helm upgrade kafka-cluster strimzi/strimzi-kafka-operator -f values.yaml -n kafka



### Scaling Kafka Brokers

The number of Kafka brokers (replicas) is configured in values.yaml:

  kafka:
    replicas: 2

TO SCALE:

Change the replicas value.

Re-run the Helm upgrade command above.

Kafka will gracefully roll the brokers in and out.

### Why 2 brokers?

Ensures availability during single node failure.

With a replication factor of 2, messages are fault-tolerant.

✅ Topic Strategy :

  The posts topic is defined with:

    partitions: 3
    replicas: 2

### Reasoning for Topic Strategy

    3 partitions: enables concurrent message consumption across 3 consumers for better throughput.

    2 replicas: guarantees data availability even if one broker goes down.

### Helm Installation (Centralized)

helm repo add strimzi https://strimzi.io/charts/
helm repo update
helm install strimzi-kafka-operator strimzi/strimzi-kafka-operator -n kafka --create-namespace

Deploy Kafka Cluster

helm install kafka-cluster ./charts/kafka-cluster -n kafka -f values.yaml



### Containerization best practices
    Dockerfiles use minimal base images (python:3.9-slim)

    .dockerignore excludes unused build files

    Environment variables drive dynamic configs (Kafka brokers, topic)

    📜 Helm Installation (Centralized)
    Add Strimzi Helm Repo:

    helm repo add strimzi https://strimzi.io/charts/
    helm repo update


    bash
    Copy
    Edit
    helm install strimzi-kafka-operator strimzi/strimzi-kafka-operator -n kafka --create-namespace
    Deploy the cluster:

    bash
    Copy
    Edit
    helm install kafka-cluster ./charts/kafka-cluster -n kafka -f values.yaml
    Replace ./charts/kafka-cluster with the path to your Helm chart or use Helm template if you don't have a custom one.

    🔥 Optional: Observability
    To monitor Kafka:

    Use Prometheus + Grafana stack.

    Strimzi exposes metrics via Kafka metrics exporter.

    Install via Helm or customize Grafana dashboards from Strimzi docs.
