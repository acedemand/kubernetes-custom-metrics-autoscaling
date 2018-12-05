# kubernetes-custom-metrics-autoscaling

This repo is a step by step guide about how to autoscale Kubernetes pods according to custom metric values. Manifests are tested in GKE. 

## Install Helm and Tiller

```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-linux-amd64.tar.gz
tar -zxvf helm-v2.11.0-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
kubectl create clusterrolebinding user-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
kubectl create serviceaccount tiller --namespace kube-system
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account=tiller
helm update
```

## Install Prometheus

```
kubectl create namespace monitoring
helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
helm install coreos/prometheus-operator --name prometheus-operator --namespace monitoring
helm install coreos/kube-prometheus --name kube-prometheus --namespace monitoring
```

## Install Prometheus Adapter

```
helm install stable/prometheus-adapter --name prometheus-adapter --set prometheus.url=http://kube-prometheus.monitoring.svc --namespace monitoring
```

## Deploy Sample Application

```
kubectl create ns test
kubectl apply -f sample-app-deployment.yaml
kubectl apply -f sample-app-service.yaml
```

## Create Service Monitor

```
kubectl apply -f service-monitor.yaml
```

## Create Horizontal Pod Autoscaler

```
kubectl apply -f hpa.yaml
```

## Test System

Do some load testing for this sample application and watch pods, hpa in "test" namespace.

## References

[1] - https://github.com/DirectXMan12/k8s-prometheus-adapter

[2] - https://github.com/kubeless/kubeless/tree/master/manifests/autoscaling
