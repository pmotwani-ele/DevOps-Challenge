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
  1)   Dockerfiles use minimal base images (python:3.9-slim)

  2)   .dockerignore excludes unused build files

  3)   Environment variables drive dynamic configs (Kafka brokers, topic)

### local development

Local Development with Docker Compose
To test the Kafka ecosystem locally without Kubernetes:

📦 Prerequisites:
Docker & Docker Compose installed

🚀 Start the services:

docker-compose up --build

This will bring up:

- Kafka + Zookeeper

- producer and consumer apps

Both will connect to the Kafka service (kafka:9092) and use the topic posts.

🔄 Developer Workflow:

1) Make changes in ./producer or ./consumer

2) Docker Compose will rebuild the container on next run

3) No need to push images or redeploy to Kubernetes


## 📈 Optional: Kafka Monitoring with Prometheus + Grafana

Kafka and Zookeeper expose Prometheus metrics via Strimzi.

To monitor:

1. Enable metrics in `values.yaml`
2. Install Prometheus + Grafana using Bitnami Helm chart
3. Create a `ServiceMonitor` for Kafka exporters
4. Access Grafana and import Strimzi dashboards

This adds visibility into Kafka partitions, brokers, topic throughput, and lag.


## Steps Observability ##

1) Enabled metrics for zookeeper and kafka for metrics in Helm charts
metrics:
  enabled: true

2) Install the charts here

  helm repo add bitnami https://charts.bitnami.com/bitnami
  helm repo update

  helm install monitoring bitnami/kube-prometheus -n monitoring --create-namespace

3) Create a service monitor.yaml ( see under Observability)

4) Port forward Grafana or Prometheus like below
kubectl port-forward --namespace monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090

Access http://127.0.0.1:9090/ locally to access prometheus


## what is not done in the favor of time and can be improved in this setup ####

Consider using an Ingress Load Balancer for the consumer and producer backend services to improve availability and manageability.

Similar to Kafka, Helm charts can also be utilized for deploying backend services, which helps standardize and simplify deployments.

CI/CD pipelines for deployment and PR reviews are currently missing. Given project timelines, this was deprioritized, but in a real-world scenario, implementing automated CI pipelines would be essential.

For scaling infrastructure in cloud environments, tools like Karpenter or the Cluster Autoscaler can be used for dynamic node provisioning. In more complex or production-scale environments, a combination of VPA (Vertical Pod Autoscaler) and HPA (Horizontal Pod Autoscaler) would be appropriate. (This setup was on Minikube, so scaling was limited.)

For Kafka, a managed service like AWS MSK can be leveraged in real-time environments to enhance scalability, simplify management, and reduce operational overhead.

As for Infrastructure as Code (IaC), Helm was primarily used here due to its simplicity and the availability of external charts. However, in production, creating custom Helm charts or using tools like Terraform or Terragrunt (or a combination) is recommended. The goal was to focus on templating and automation.
