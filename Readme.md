# Monitor a Kubernetes (EKS) Cluster with Prometheus & Grafana

This guide installs Helm and eksctl, deploys the `kube-prometheus-stack` Helm chart, and exposes Prometheus and Grafana through LoadBalancers.

## Prerequisites

- A running Kubernetes cluster (for example, Amazon EKS)
- `kubectl` configured to talk to the cluster

---

## 1. Install Helm

**Option A: Latest version**

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

**Option B: Specific version (e.g. v3.8.2)**

```bash
DESIRED_VERSION=v3.8.2 bash get_helm.sh
```

**Verify the installation**

```bash
helm version
```

---

## 2. Install eksctl

```bash
sudo apt update && sudo apt upgrade -y
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

## 3. Add Helm Chart Repositories

```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 4. Create the Prometheus Namespace

```bash
kubectl create namespace prometheus
kubectl get ns
```

---

## 5. Install Prometheus Using Helm

```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```

Verify the pods and services:

```bash
kubectl get pods -n prometheus
kubectl get svc -n prometheus
```

---

## 6. Expose Prometheus and Grafana Externally

There are two ways to expose the services:

1. **NodePort**
2. **LoadBalancer** (used in this guide)

### Expose Prometheus

Edit the service and change `type: ClusterIP` to `type: LoadBalancer`, then save:

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

Or do it in one command:

```bash
kubectl patch svc stable-kube-prometheus-sta-prometheus -n prometheus -p '{"spec": {"type": "LoadBalancer"}}'
```

Check the service:

```bash
kubectl get svc -n prometheus
```

A load balancer is now provisioned for Prometheus. Access it using the external address on port **9090**.

### Expose Grafana

Change the Grafana service type from `ClusterIP` to `LoadBalancer` the same way:

```bash
kubectl edit svc stable-grafana -n prometheus
```

Or:

```bash
kubectl patch svc stable-grafana -n prometheus -p '{"spec": {"type": "LoadBalancer"}}'
```

Check the service:

```bash
kubectl get svc -n prometheus
```

Copy the Grafana `EXTERNAL-IP` / hostname and open it in your browser. You will be prompted for a username and password.

---

## 7. Get Grafana Login Credentials

**Username**

```bash
kubectl get secret stable-grafana -n prometheus \
  -o jsonpath="{.data.admin-user}" | base64 --decode ; echo
```

**Password**

```bash
kubectl get secret stable-grafana -n prometheus \
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Default login:

| Field    | Value                                     |
|----------|-------------------------------------------|
| Username | `admin`                                   |
| Password | *(retrieve using the command above)*      |

> ⚠️ Never commit passwords or secrets to your repository.

---

## 8. Configure Dashboards

Log in to Grafana and open the pre-built Kubernetes dashboards to visualize your cluster's health, nodes, pods, and resource usage.

---

## Cleanup

```bash
helm uninstall stable -n prometheus
kubectl delete namespace prometheus
```
