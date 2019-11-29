
Install istio with zipkin using helm
```
cd istio-1.3.4

helm install install/kubernetes/helm/istio \
--name istio \
--namespace istio-system \
--values install/kubernetes/helm/istio/values-istio-demo.yaml \
--set values.tracing.enabled=true \
--set values.tracing.provider=zipkin
```

View service graph

```
istioctl dashboard kiali
```

Prometheus monitoring

istioctl dashboard prometheus

Use below queries
```
sum(rate(container_cpu_usage_seconds_total{container_name!="", namespace="default"}[5m])) by (pod_name)
```

```
sum(rate(container_cpu_usage_seconds_total{container_name!="", namespace="default"}[5m])) by (container_name)
```
