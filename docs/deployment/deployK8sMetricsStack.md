---
title: Kubernetes Cluster Metrics Stack
---
# Kubernetes Cluster Metrics Stack

This will cover deploying a metrics stack for your kubernetes cluster into the `metrics` namespace. The files referenced correspond to pingidentity-devops-getting-started/20-kubernetes/20-cluster-metrics-stack


## Prerequisites and References

You will need:
  * helm
  * envsubst - optional but useful.
  * _cluster admin privileges_
  * knowledge of kubernetes 

This work leverages:
* [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
* [Telegraf Operator](https://github.com/influxdata/helm-charts/tree/master/charts/telegraf-operator)
### What You'll Do

1. [Deploy Prometheus Operator](#deploy-prometheus-operator)
1. [Deploy Telegraf operator](#deploy-telegraf-operator)
1. [Annotate Ping deployment for metrics to be collected](#annotate-ping-deployment)
1. [View metrics in Grafana](#view-metrics-in-grafana)

## Deploy Prometheus Operator

Start by exporting some variables in your terminal:
```
export MY_RELEASE_NAME=metrics
export MY_DOMAIN=ping-devops.com
```

Add the helm chart repo:

  ```
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  ```

`20-cluster-metrics-stack/prometheus-operator/values.yaml` is slightly modified from the prometheus-operator default `values.yaml`. You can make any other adjustments as needed, refer to the [prometheus-operator helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) for reference. 


Our edits to this yaml include:
- enabling ingress (tested with nginx ingress controller) - uses ingressclass=nginx-public. You will likely need to change this in three sections (alertmanager, prometheus, grafana)
- adding a `scrape-config` for Telegraf
- adding persistence to Prometheus

When ready, deploy the release and dashboards: 
  
  ```
  envsubst '$MY_RELEASE_NAME $MY_DOMAIN' < prometheus-operator/values.yaml > prometheus-operator/values.yaml.final

  helm upgrade --install ${MY_RELEASE_NAME} prometheus-community/kube-prometheus-stack -f <path/to>/20-kubernetes/20-cluster-metrics-stack/prometheus-operator/values.yaml.final -n metrics --version 14.5.0

  kubectl apply -f prometheus-operator/dashboards -n metrics
  ```

## Deploy Telegraf Operator

The Telegraf Operator watches for pods with specific annotation and injects sidecars for reading and exposing metrics to prometheus.

Add the helm chart repo: 

  ```
  helm repo add influxdata https://helm.influxdata.com/
  ```

Edit `telegraf/values.yaml` only if you want to add another [output](https://github.com/influxdata/telegraf/tree/master/plugins/outputs).

Deploy the release

  ```
  helm upgrade --install ${MU_RELEASE_NAME} influxdata/telegraf-operator -f telegraf/values.yaml -n metrics --version 1.1.5
  ```

## Annotate Ping Deployment

This sample `ping/values.yaml` may be outdated or you may have your own deployment already. If so, the section to copy and add to your own deployment is the `<product-name>.workload.annotations` for each product runtime you use. 

To use this default:

  ```
  kubectl create secret generic devops-secret --from-literal=PING_IDENTITY_DEVOPS_USER="${PING_IDENTITY_DEVOPS_USER}" --from-literal=PING_IDENTITY_DEVOPS_KEY="${PING_IDENTITY_DEVOPS_KEY}" --from-literal=PING_IDENTITY_ACCEPT_EULA="YES"

  helm repo add pingidentity https://helm.pingidentity.com/

  helm upgrade --install ${MY_RELEASE_NAME} pingidentity/ping-devops --version 0.6.2 -f ping/values.yaml
  ```

## View Metrics in Grafana

Browse to your ingress host for grafana or use kubectl port-forward

  ```
  kubectl port-forward svc/${MY_RELEASE_NAME}-grafana -n metrics 8080:80
  ```

Log in with

User: admin
Password: 2FederateM0re

Hover on the Dashboards icon > Manage. Pick a dashboard. 
