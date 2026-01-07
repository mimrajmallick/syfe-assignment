<!-- README test change -->
 Syfe DevOps Intern Assignment

Project Overview : 
Production-grade WordPress deployment on Kubernetes with OpenResty Nginx, MySQL, and comprehensive monitoring.

Assignment Requirements Checklist :
REQUIREMENT #1: Production WordPress on Kubernetes

PersistentVolumeClaims and PersistentVolumes - Configured with ReadWriteMany for scaling

DockerFiles - MySQL, WordPress, and OpenResty Nginx with Lua

OpenResty with exact configure options - Compiled with specified build flags

Helm chart deployment - Single command: helm install wordpress-stack ./helm-charts/wordpress

Cleanup - helm uninstall wordpress-stack

REQUIREMENT #2: Monitoring & Alerting

Prometheus/Grafana configuration - Alert rules for production monitoring

Pod CPU utilisation - HighPodCPU alert configured

Nginx monitoring - Total Request Count & 5xx errors documented

Complete metrics documentation - METRICS.md with all required metrics

Architecture :
text
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                    │
│                                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌───────────┐  │
│  │    Nginx    │────▶│  WordPress  │────▶│   MySQL   │  │
│  │ (OpenResty) │     │  (Apache)   │     │           │  │
│  └─────────────┘     └─────────────┘     └───────────┘  │
│         │                    │                    │      │
│         └────────────────────┴────────────────────┘      │
│                          PVCs                            │
│                    (ReadWriteMany)                       │
│                                                         │
│  ┌─────────────────────────────────────────────────────┐│
│  │               Monitoring Stack                      ││
│  │          Prometheus + Grafana + Alerts              ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
Quick Start :
Prerequisites
Docker Desktop

Minikube

Helm 3.x

kubectl

1. Start Minikube
bash
minikube start --memory=4096 --cpus=3
minikube addons enable ingress
minikube addons enable metrics-server
2. Build Docker Images
bash
# Use Minikube's Docker daemon
eval $(minikube docker-env)

# Build all images
docker build -t my-mysql:1.0 -f dockerfiles/mysql/Dockerfile dockerfiles/mysql/
docker build -t my-wordpress:1.0 -f dockerfiles/wordpress/Dockerfile dockerfiles/wordpress/
docker build -t my-nginx:1.0 -f dockerfiles/nginx/Dockerfile dockerfiles/nginx/
c
4. Access WordPress
bash
# Method 1: Port-forward (recommended)
kubectl port-forward deployment/wordpress-stack-wordpress 8080:80
# Open http://localhost:8080

# Method 2: Minikube service
minikube service wordpress-stack-nginx-service --url
5. Deploy Monitoring (Optional)
bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --create-namespace
Key Implementation Details :
OpenResty with Lua
The Nginx reverse proxy uses OpenResty compiled with exact configuration options as specified:

bash
./configure \
    --prefix=/opt/openresty \
    --with-pcre-jit \
    --with-ipv6 \
    --without-http_redis2_module \
    --with-http_iconv_module \
    --with-http_postgres_module \
    -j8
Persistent Storage Strategy
MySQL: ReadWriteOnce PVC for database persistence

WordPress: ReadWriteMany PVC for horizontal pod scaling

Monitoring: Persistent storage for Prometheus metrics

Helm Chart Structure
text
helm-charts/wordpress/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Configuration values
└── templates/              # Kubernetes manifests
    ├── mysql-deployment.yaml
    ├── mysql-pvc.yaml
    ├── mysql-service.yaml
    ├── nginx-deployment.yaml
    ├── nginx-service.yaml
    ├── wordpress-deployment.yaml
    ├── wordpress-pvc.yaml
    └── wordpress-service.yaml
Monitoring Configuration
Prometheus Alerts: CPU utilization, 5xx errors, pod health

Grafana Dashboards: Pre-configured for WordPress stack

Custom Metrics: Nginx request counts, error rates, application metrics

Verification & Evidence :
Current Deployment Status
bash
$ kubectl get pods
NAME                                        READY   STATUS    RESTARTS   AGE
wordpress-stack-mysql-7b45c7657c-hcfxr      1/1     Running   0          45m
wordpress-stack-nginx-6ffc546568-pgjwd      1/1     Running   6          6h
wordpress-stack-wordpress-c974f84bd-b5q26   1/1     Running   0          42m
wordpress-stack-wordpress-c974f84bd-h9sgn   1/1     Running   0          31m

$ kubectl get pvc
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES
wordpress-stack-mysql-pvc   Bound    pvc-17bd7e8b-0c44-4116-9851-68b60f41be54   10Gi       RWO
wordpress-stack-wp-pvc      Bound    pvc-750d605b-1f01-4ed2-a40f-4fe277463deb   10Gi       RWX  # ReadWriteMany

$ kubectl exec deployment/wordpress-stack-nginx -- curl http://localhost/health
healthy
WordPress Access Proof
WordPress successfully deployed and accessible:

URL: http://localhost:8080/wp-admin/install.php

Status: Installation page loading successfully

## WordPress Access Proof

![WordPress Installation Step 1](images/Screenshot%202026-01-07%20at%204.14.16%E2%80%AFPM.png)
![WordPress Installation Step 2](images/Screenshot%202026-01-07%20at%204.14.32%E2%80%AFPM.png)


Monitoring Evidence
bash
# CPU Alert Configuration
$ grep -A2 "alert: HighPodCPU" monitoring/alerts.yaml
- alert: HighPodCPU
  expr: (sum(rate(container_cpu_usage_seconds_total{container="wordpress"}[5m])) by (pod) / (sum(container_spec_cpu_quota{container="wordpress"} / 100000) by (pod))) * 100 > 80

# 5xx Error Monitoring
$ grep -i "5xx" METRICS.md
- **Total 5xx Errors**: `nginx_http_requests_total{status=~"5.."}`

Project Structure : 

text
syfe-assignment/
├── dockerfiles/                    # All Dockerfiles
│   ├── mysql/                      # MySQL with custom configuration
│   ├── nginx/                      # OpenResty with Lua support
│   └── wordpress/                  # WordPress with Apache
├── helm-charts/                    # Complete Helm deployment
│   └── wordpress/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/              # Kubernetes manifests
├── monitoring/                     # Prometheus alert rules
│   └── alerts.yaml                 # CPU and 5xx error alerts
├── README.md                       # This documentation
├── METRICS.md                      # Complete metrics documentation
└── screenshots/                    # Verification evidence
    └── wordpress-success.png       # WordPress installation proof

Troubleshooting :

Common Issues
ImagePullBackOff

bash
# Rebuild images in Minikube's Docker
eval $(minikube docker-env)
docker build -t my-wordpress:1.0 -f dockerfiles/wordpress/Dockerfile dockerfiles/wordpress/
PVC Stuck in Pending

bash
# Check storage class
kubectl get storageclass
# For Minikube testing, use hostPath
Service Unreachable

bash
# Always works:
kubectl port-forward deployment/wordpress-stack-wordpress 8080:80
Resource Constraints

bash
# Reduce replicas in values.yaml
wordpress:
  replicaCount: 1
nginx:
  replicaCount: 1
Verification Commands
bash
# Verify all components
kubectl get all
kubectl describe pvc wordpress-stack-wp-pvc | grep -i "access modes"
helm list
docker images | grep -E "(my-nginx|my-wordpress|my-mysql)"
Documentation :
METRICS.md: Complete metrics specification for WordPress, Apache, and Nginx

Helm Charts: Production-ready deployment templates

Dockerfiles: Optimized container configurations

Cleanup :
bash
# Uninstall WordPress stack
helm uninstall wordpress-stack

# Uninstall monitoring
helm uninstall monitoring -n monitoring

# Delete Minikube cluster
minikube delete

Resources :
[Kubernetes Documentation](https://kubernetes.io/docs/home/)

[Helm Documentation](https://helm.sh/docs/)

[Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)

[OpenResty Documentation](https://openresty.org/en/)

Final Status :

All assignment requirements successfully implemented and verified.

[GitHub Repository](https://github.com/mimrajmallick/syfe-assignment)

[WordPress Access](http://localhost:8080/wp-admin/install.php)

Ready for production deployment with:

Scalable architecture (ReadWriteMany PVC)

Comprehensive monitoring

Infrastructure as Code (Helm charts)

Production-grade configurations

Contact Information
Name: Samim Mallick
Phone: 9609118970
Email: mimrajmallick79@gmail.com

For any questions or assistance with this deployment, feel free to reach out! 

[def]: screenshots/wordpress-install-2.png. why this is not push to github
