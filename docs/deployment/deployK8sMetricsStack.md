---
title: Kubernetes Cluster Metrics Stack
---
# Kubernetes Cluster Metrics Stack

This will cover deploying a metrics stack for your kubernetes cluster into the `metrics` namespace. 
Focus

## Prerequisites and References

* envsubst
* helm
* [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
* [Telegraf Operator](https://github.com/influxdata/helm-charts/tree/master/charts/telegraf-operator)

### What You'll Do

1. Deploy Prometheus Operator(#deploy-prometheus-operator)
1. Deploy Telegraf operator
1. Annotate Ping deployment for metrics to be collected
1. View metrics in Grafana

## Deploy Prometheus Operator

Add the helm chart repo:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

The [values.yaml](../../20-kubernetes/20-cluster-metrics-stack/prom-operator/values.yaml) used has been slightly modified from the default. You can make any other adjustments as needed, refer to the [prometheus-operator helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) for reference. 

Our edits to this yaml include:
- enabling ingress (tested with nginx ingress controller)
- adding a `scrape-config` for Telegraf
- adding persistence to Prometheus

To deploy the chart: 
<!-- EDIT THIS -->
helm upgrade --install metrics prometheus-community/kube-prometheus-stack -f <path/to>/20-kubernetes/20-cluster-metrics-stack/prom-operator/values.yaml -n metrics --version 14.5.0
