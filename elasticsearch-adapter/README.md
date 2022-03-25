# Elasticsearch metrics adapter demo deployment manifests

The files in this directory can be used to deploy the Elasticsearch metrics adapter.

* `01_config.yaml`: Elasticsearch metrics adapter configuration file to expose to the K8S autoscaling controller metrics from both Elasticsearch and Prometheus.
* `02_deployment.yaml`: Elasticsearch metrics adapter deployment file, (Role)Bindings, APIService and Deployment.
* `03_simple-service.hpa.yaml`: Sample HorizontalPodAutoscaler resource prefixed with `prometheus.metrics.`.
