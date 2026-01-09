# Kubernetes Cluster on AWS EC2 Using kubeadm with Observability Stack

## Overview
This project demonstrates creation of a Kubernetes cluster on AWS EC2 instances using kubeadm, deployed across multiple Availability Zones in us-west-1 with the control plane in us-west-1a and a worker node in us-west-1b.

Kubernetes components are manually configured on Ubuntu EC2 instances with Calico or Flannel for pod networking. A containerized Node.js application is exposed via NGINX Ingress and instrumented using OpenTelemetry, with traces visualized in Jaeger. Prometheus and Grafana, deployed via Helm, provide cluster metrics and monitoring dashboards.


---

## AWS Architecture Diagram

<p align="center">
  <img src="./diagram/Architecture Diagram.png" alt="Architecture Diagram" width="900">
</p>

---

## K8s Architecture

### Control Plane (Master Node)
- API Server  
- Scheduler  
- Controller Manager  
- etcd  

### Worker Nodes (Data Plane)
- kubelet  
- kube-proxy  
- container runtime (containerd)  

### Networking
- Container Network Interface (CNI): **Calico or Flannel**
- Enables pod-to-pod and node-to-node communication

### kubectl
- Command-line tool for cluster administration

---

## AWS Infrastructure

### VPC Configuration
- VPC CIDR: `10.0.0.0/16`
- Public Subnets:
  - `10.0.1.0/24` (us-west-1a)
  - `10.0.2.0/24` (us-west-1b)
- Internet Gateway
- Public Route Tables

### Security Groups
Separate security groups for control plane and worker nodes with required Kubernetes, NodePort, and Calico ports enabled.

---

## EC2 Instances

### Control Plane
- Ubuntu Linux
- Instance Type: `t3.medium`
- Disk: 20 GiB (gp3)
- Subnet: Public Subnet A
- Security Group: `k8s-control-plane-sg`

### Worker Node
- Ubuntu Linux
- Instance Type: `t3.medium`
- Disk: 20 GiB (gp3)
- Subnet: Public Subnet B
- Security Group: `k8s-worker-node-sg`

---

## Kubernetes Cluster Setup (kubeadm)

Steps performed on **both control plane and worker nodes**:
- Disable swap
- Load required kernel modules
- Configure sysctl parameters
- Install and configure containerd
- Install kubeadm, kubelet, and kubectl

### Control Plane Initialization
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

Configure kubectl access and install CNI (Calico or Flannel).

### Worker Node Join
```bash
sudo kubeadm join <CONTROL-PLANE-IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

---

## cert-manager Installation
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.19.2/cert-manager.yaml
```

---

## OpenTelemetry Setup

### Components
- Operator: `opentelemetry-operator-system`
- Collector + Instrumentation: `opentelemetry`

### Operator Installation
```bash
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml
```

### Collector Deployment
- Create `otel-collector.yaml`
- Deploy into `opentelemetry` namespace

---

## Node.js Application Deployment

- Containerized Node.js app
- Docker image pushed to DockerHub
- Kubernetes resources:
  - Deployment
  - Service
  - NGINX Ingress Controller

Ingress exposes the application via **NodePort**.

---

## OpenTelemetry Instrumentation

- Auto-injects Node.js OpenTelemetry SDK
- Applies at pod startup
- Requires deployment restart after instrumentation is applied

---

## Distributed Tracing with Jaeger

- Jaeger deployed using `jaegertracing/all-in-one`
- OpenTelemetry Collector exports traces to Jaeger
- Jaeger UI exposed via NodePort

---

## Helm Installation
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

---

## Prometheus & Grafana Monitoring

### kube-prometheus-stack
Installed via Helm in `monitoring` namespace:
- Prometheus
- Grafana
- Alertmanager
- Exporters

Grafana exposed via NodePort.

---

## Grafana Dashboards

Prebuilt dashboards include:
- Kubernetes / Compute Resources / Node (Pods)
- Kubernetes / Compute Resources / Cluster

These provide real-time cluster and workload metrics.

---

## Outcome

- Fully functional Kubernetes cluster on AWS EC2
- Node.js application deployed with ingress
- End-to-end observability using OpenTelemetry
- Tracing with Jaeger
- Metrics and dashboards with Prometheus & Grafana

---
