# [Elasticsearch Metrics Adapter](https://github.com/elastic/elasticsearch-k8s-metrics-adapter) demo manifests

## Prerequisites

* This setup has been tested on a 6 nodes (`n1-standard-8`) K8S 1.22 cluster.
* The `${DEMO}` environment variable must hold the path to this file's directory.

1. Deploy ECK:
`helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace --values=${DEMO}/eck/profile.yaml`

2. Deploy the prometheus operator:
`helm install prometheus prometheus-community/prometheus -n prometheus-monitoring --create-namespace`

3. Deploy the sample application:
`kubectl apply -f ${DEMO}/demo_app/simple-go-service-container.yaml`

4. Prometheus validation:
check that metrics are available in Prometheus using the port-forwarder: `k port-forward svc/prometheus-server 9090:80 -n prometheus-monitoring` (for example using the request `go_goroutines{pod="elastic-operator-0"}`)

## Demo

See [`demo.md`](demo.md)

## Slides

See [`slides.pdf`](slides.pdf)
