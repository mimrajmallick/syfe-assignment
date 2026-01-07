 <!-- README test change -->
Syfe DevOps Intern Assignment
Project Overview

Production-grade WordPress deployment on Kubernetes using OpenResty (Nginx), MySQL, and a complete monitoring stack with Prometheus and Grafana.

This project demonstrates Kubernetes best practices, infrastructure-as-code using Helm, persistent storage management, and production-level monitoring and alerting.

Assignment Requirements Checklist
Requirement #1: Production WordPress on Kubernetes

PersistentVolumes and PersistentVolumeClaims configured

WordPress PVC: ReadWriteMany (horizontal scaling)

MySQL PVC: ReadWriteOnce

Custom Dockerfiles:

MySQL

WordPress (Apache)

OpenResty Nginx with Lua

OpenResty compiled with exact required configuration options

Helm-based deployment

helm install wordpress-stack ./helm-charts/wordpress


Cleanup support

helm uninstall wordpress-stack

Requirement #2: Monitoring & Alerting

Prometheus & Grafana setup

Alert rules for:

High Pod CPU utilization

Nginx 5xx errors

Nginx metrics:

Total request count

Error rate

Complete metrics documentation in METRICS.md

Architecture
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                  │
│                                                       │
│  ┌─────────────┐     ┌─────────────┐     ┌───────────┐│
│  │    Nginx    │────▶│  WordPress  │────▶│   MySQL   ││
│  │ (OpenResty) │     │  (Apache)   │     │           ││
│  └─────────────┘     └─────────────┘     └───────────┘│
│         │                    │                    │   │
│         └────────────────────┴────────────────────┘   │
│                         PVCs                           │
│                    (ReadWriteMany)                     │
│                                                       │
│  ┌───────────────────────────────────────────────────┐│
│  │               Monitoring Stack                    ││
│  │        Prometheus + Grafana + Alerts               ││
│  └───────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘

Quick Start
Prerequisites

Docker Desktop

Minikube

Helm 3.x

kubectl

1. Start Minikube
minikube start --memory=4096 --cpus=3
minikube addons enable ingress
minikube addons enable metrics-server

2. Build Docker Images

Use Minikube’s Docker daemon:

eval $(minikube docker-env)


Build images:

docker build -t my-mysql:1.0 -f dockerfiles/mysql/Dockerfile dockerfiles/mysql/
docker build -t my-wordpress:1.0 -f dockerfiles/wordpress/Dockerfile dockerfiles/wordpress/
docker build -t my-nginx:1.0 -f dockerfiles/nginx/Dockerfile dockerfiles/nginx/

3. Deploy WordPress Stack
helm install wordpress-stack ./helm-charts/wordpress

4. Access WordPress

Method 1 (Recommended): Port Forward

kubectl port-forward deployment/wordpress-stack-wordpress 8080:80


Open:
http://localhost:8080

Method 2: Minikube Service

minikube service wordpress-stack-nginx-service --url

5. Deploy Monitoring (Optional)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --create-namespace

Key Implementation Details
OpenResty with Lua

OpenResty is compiled with the following configuration:

./configure \
  --prefix=/opt/openresty \
  --with-pcre-jit \
  --with-ipv6 \
  --without-http_redis2_module \
  --with-http_iconv_module \
  --with-http_postgres_module \
  -j8

Persistent Storage Strategy

MySQL: ReadWriteOnce PVC (database persistence)

WordPress: ReadWriteMany PVC (horizontal scaling)

Monitoring: Persistent storage for Prometheus metrics

Helm Chart Structure
helm-charts/wordpress/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── mysql-deployment.yaml
    ├── mysql-pvc.yaml
    ├── mysql-service.yaml
    ├── nginx-deployment.yaml
    ├── nginx-service.yaml
    ├── wordpress-deployment.yaml
    ├── wordpress-pvc.yaml
    └── wordpress-service.yaml

Verification & Evidence
Current Deployment Status
kubectl get pods

kubectl get pvc

kubectl exec deployment/wordpress-stack-nginx -- curl http://localhost/health


Expected output:

healthy

WordPress Access Proof

URL:
http://localhost:8080/wp-admin/install.php

Status:
Installation page loads successfully

Screenshots are available in the images/ or screenshots/ directory.

Monitoring Evidence
CPU Alert Configuration
grep -A2 "alert: HighPodCPU" monitoring/alerts.yaml

5xx Error Monitoring
grep -i "5xx" METRICS.md

Project Structure
syfe-assignment/
├── dockerfiles/
│   ├── mysql/
│   ├── nginx/
│   └── wordpress/
├── helm-charts/
│   └── wordpress/
├── monitoring/
│   └── alerts.yaml
├── README.md
├── METRICS.md
└── screenshots/

Troubleshooting
ImagePullBackOff
eval $(minikube docker-env)
docker build -t my-wordpress:1.0 -f dockerfiles/wordpress/Dockerfile dockerfiles/wordpress/

PVC Stuck in Pending
kubectl get storageclass


Use hostPath for Minikube testing.

Service Unreachable
kubectl port-forward deployment/wordpress-stack-wordpress 8080:80

Cleanup
helm uninstall wordpress-stack
helm uninstall monitoring -n monitoring
minikube delete

Resources :
[Kubernetes Documentation](https://kubernetes.io/docs/home/)

[Helm Documentation](https://helm.sh/docs/)

[Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)

[OpenResty Documentation](https://openresty.org/en/)

Final Status

All assignment requirements successfully implemented and verified.

GitHub Repository: https://github.com/mimrajmallick/syfe-assignment

WordPress Access: http://localhost:8080/wp-admin/install.php

Contact Information

Name: Samim Mallick
Phone: 9609118970
Email: mimrajmallick79@gmail.com

For any questions or clarification, feel free to reach out.
