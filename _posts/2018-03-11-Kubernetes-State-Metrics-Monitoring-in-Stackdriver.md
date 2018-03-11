---
layout: post
title: Kubernetes State Metrics Monitoring in Stackdriver
---

This is a guide on how to export metrics about the state of a Kubernetes cluster hosted on GKE to Stackdriver. We will be deploying [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) and [prometheus-to-sd](https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/prometheus-to-sd) applications to a Kubernetes cluster to scape data from the Kubernetes API and export it to Stackdriver. Stackdriver out of the box will only monitor node level metrics, such as CPU and memory usage, and container level metrics. If you need more detailed information about Kubernetes level metrics such as the number of healthy pods in a service or alerting on container restarts you must scrape and export them to Stackdriver as custom metrics. 

### Deploying to Kubernetes

We will be taking advantage of open-sourced helm charts for speed and reproduceability. [Helm](https://helm.sh/) is quickly becoming the de-facto tool for deploying applications to Kubernetes and it's [public repository](https://github.com/kubernetes/charts) has a ton a useful applications that are preconfigured to work out of the box. This guide will be assuming familiarity with Helm and general concepts regarding deployments to Kubernetes.

### Why export to Stackdriver?

At smash.gg we are already using Stackdriver for monitoring most resources in our GCP project through dashboards as well as automated alerting to slack channels. As a startup we'd like to use a managed service for monitoring so it's one less application in our stack that we have to deploy, configure, and ensure uptime on. We needed alerting on the failures of any [Kubernetes CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) we were running and this seemed to be the quicklest and most elegant solution.

### [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)

kube-state-metrics is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. (See examples in the Metrics section below.) It is not focused on the health of the individual Kubernetes components, but rather on the health of the various objects inside, such as deployments, nodes and pods.

### [prometheus-to-sd](https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/prometheus-to-sd)

prometheus-to-sd is a simple component that can scrape metrics stored in prometheus text format from one or multiple components and push them to the Stackdriver. We will use this to scrape from kube-state-metrics and export to Stackdriver as a custom metric.

### Code

We deploy all of our helm charts through a Jenkins server but for this guide we will be showing steps from any machine that has the following:
- Ability to authenticate to the GKE cluster through the `gcloud` cli tool
- Kubectl installed if it isn't already from the gcloud cli
- Helm client installed on your machine

First we'll start with authenticating to the cluster:
```bash
gcloud container clusters get-credentials $CLUSTER \
    --zone $ZONE \
    --project $PROJECT
```

Next we will initialize helm. This will install [Tiller](https://docs.helm.sh/using_helm/#installing-tiller), the Helm server, on the cluster if it doesn't already exist.
```bash
# If Tiller is not installed:
helm init

# If you are running this on a containerized build environment 
# (such as a Jenkins agent) and know Tiller is already installed:
helm init --client-only
```

Create the Kubenetes namespace. We will be using "monitoring" for this guide but you can use whatever you see fit.
```bash
kubectl create ns monitoring
```

Deploy the kube-state-metrics application:
```bash
helm install \
    --namespace monitoring \
    kube-state-metrics stable/kube-state-metrics
```

You will need the endpoint of the kube-state-metrics for pointing prometheus-to-sd to it in the next step. If you use the command above the endpoint for reaching the service will be:

`ENDPOINT=http://kube-state-metrics-kube-state-metrics.monitoring.svc.cluster.local:8080` 

And you can then install prometheus-to-sd:
```bash
helm install \
    --namespace monitoring \
    --set metricsSources.kube-state-metrics=$ENDPOINT
    kube-state-metrics stable/kube-state-metrics
```

And that's it! You can verify this configuration is working by going to Stackdriver > Metric Explorer and start typing "custom/kube-state-metrics/" and the search bar should fuzzy-find all of your new kubernetes metrics. You can create dashboards and alerts just like any other metrics with this new data source. 
