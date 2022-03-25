# Demo notes

Refer to the [demo architecture slide](slides.pdf) for more details about the components involved in this demo.

## ECK Quickstart

1. Deploy a simple `Beats`+`Kibana`+`Elasticsearch` deployment to collect Prometheus metrics.
[`kubectl apply -f ${DEMO}/kibana_es_beats.yaml`](kibana_es_beats.yaml)

It is advised to use a tool like `kubens` to move from your current default namespace to `elastic-monitoring`:

```bash
> kubens elastic-monitoring
Context "gke_elastic-europe-west1_michael-demo" modified.
Active namespace is "elastic-monitoring".`
```

2. Get the `elastic` user password: `kubectl ksd get secret monitoring-es-elastic-user -n elastic-monitoring -o yaml`
3. Use `kubectl port-forward svc/monitoring-kb-http 5601 -n elastic-monitoring` to access Kibana from your workstation.
4. Create two vizualizations:
   1. Max of prometheus.metrics.follow_me
   2. Count of kubernetes.statefulset.replicas.desired

## Understanding the scale subresource API

1. Create an Nginx deployment
`kubectl create deploy nginx --image nginx`

2. Call the scale API
`kubectl scale -v=8 --replicas 2 deploy/nginx`

## Standard metrics

1. How de we know which service servces the standard metrics API ?
`kubectl get apiservice v1beta1.metrics.k8s.io`

2. Metrics API result: `kubectl get --raw  /apis/metrics.k8s.io/v1beta1/pods/ | jq`

## Custom metrics

### Using the Prometheus Metrics Adapter

1. How de we know which service currently serves the custom metrics API ?
`kubectl get apiservice v1beta2.custom.metrics.k8s.io`

2. Deploy the custom metrics adapter for Prometheus
`kubectl apply -f [$DEMO/prometheus-adapter/prometheus-adapter.yaml](prometheus-adapter/prometheus-adapter.yaml)`

3. How de we know which service implement this API ?
`kubectl get apiservice v1beta1.metrics.k8s.io`

4. Deploy a sample HPA resource
`k apply -f $DEMO/demo_app/simple-service.hpa.yaml`

For the moment the Elastic stack is only used to observe the demo application. Let's deploy the Elasticsearch metrics adapter.

### Using the Elasticsearch Metrics Adapter

Deploy the Elasticsearch Metrics Adapter:

1. Create the adapter configuration: [`kubectl apply -f ${DEMO}/elasticsearch-adapter/01_config.yaml`](elasticsearch-adapter/01_config.yaml)
2. Deploy the Elasticsearch Metrics Adapter: [`kubectl apply -f [${DEMO}/elasticsearch-adapter/02_deployment.yaml`](elasticsearch-adapter/02_deployment.yaml)

Note that nothing changed from the user point of view, the Elasticsearch Metrics Adapter is still using Prometheus to serve the custom metric. As a proof you can scale down Prometheus and see that it breaks the autoscaling mechanism (target is set to `unknown`):
`kubectl scale --replicas 0 deploy/prometheus-server -n prometheus-monitoring`

```bash
NAME                REFERENCE                       TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
simple-go-service   StatefulSet/simple-go-service   <unknown>/1   1         10        9          8m6s
```

Update the HPA resource to now rely on the metric stored in Elasticsearch:

[`kubectl apply -f ${DEMO}/elasticsearch-adapter/03_simple-service.hpa.yaml`](elasticsearch-adapter/03_simple-service.hpa.yaml)

After a few seconds HPA should be healthy again:

```bash
NAME                REFERENCE                       TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
simple-go-service   StatefulSet/simple-go-service   <unknown>/1   1         10        9          8m6s
simple-go-service   StatefulSet/simple-go-service   1111m/1       1         10        9          8m8s
simple-go-service   StatefulSet/simple-go-service   1/1           1         10        10         8m24s
```

### The metrics discovery API

```
k get --raw  /apis/custom.metrics.k8s.io/v1beta2/ | jq
```

```json
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "custom.metrics.k8s.io/v1beta2",
  "resources": [
    {
      "name": "jobs.batch/rpc_durations_histogram_seconds_count",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "pods/aws.networkelb.metrics.TCP_ELB_Reset_Count.sum",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "namespaces/follow_me",
      "singularName": "",
      "namespaced": false,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "pods/mysql.status.handler.rollback",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    ...
```
