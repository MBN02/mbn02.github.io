---
layout: post
title: "Monitoring with Prometheus, Grafana, and Loki"
date: 2025-11-28 11:12:00 +0800
categories: monitoring
tags: monitoring
image:
  path: /assets/img/headers/monitoring.png
---

*Having a robust monitoring stack on Kubernetes is essential to ensure the reliability, performance, and security of your applications and infrastructure.With real-time dashboards and correlated insights, the teams can optimize resource usage, enforce SLOs, and minimize downtime for modern cloud-native environments.*

{% include embed/youtube.html id='r_w-pNueZuI' %}

🎞️ [Watch Video](https://youtu.be/r_w-pNueZuI)

## Prerequisites:

- Kubernetes cluster
- Helm 3.x 0r 4.x
- kubectl configured

## Kube-prometheus-stack

#### Set Up Namespace and Helm Repositories

```sh
kubectl create namespace monitoring
```

#### Add the Prometheus Helm Repository

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update
```

```sh
helm show values prometheus-community/kube-prometheus-stack > prometheus-stack-values.yaml
```
```sh
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --version 79.8.2 -f prometheus-stack-values.yaml
```

#### Check the status of the deployment
```sh
kubectl get pods -n monitoring

kubectl describe pod prometheus-grafana-<pod-id> -n monitoring

kubectl logs prometheus-grafana-<pod-id> -n monitoring -c grafana
```

#### Accessing the grafana dashboard using portforward
```sh
kubectl port-forward svc/prometheus-grafana -n monitoring 3000:80
```

## Loki

#### Deploying Loki for Persistent Log Aggregation
```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki grafana/loki-stack --version 2.10.3 -f loki-value.yaml -n monitoring 
```

### 🔗 Reference Links:

- [Kube prometheus stack](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)

- [GitHub-Repo](https://github.com/mkbntech/monitoring-stack-kuberntes.git)
