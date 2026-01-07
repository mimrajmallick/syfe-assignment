# Metrics Documentation for WordPress Deployment

## Required Metrics (Per Assignment)

### 1. WordPress Application Metrics
- **Pod CPU Utilization**: `container_cpu_usage_seconds_total{container="wordpress"}`
- **Pod Memory Usage**: `container_memory_usage_bytes{container="wordpress"}`
- **HTTP Request Rate**: `rate(nginx_http_requests_total[5m])`

### 2. Nginx Metrics (OpenResty)
- **Total Request Count**: `nginx_http_requests_total`
- **Total 5xx Errors**: `nginx_http_requests_total{status=~"5.."}`
- **Request Rate by Status**: `rate(nginx_http_requests_total[5m])`
- **Active Connections**: `nginx_connections_active`

### 3. MySQL Database Metrics
- **Database Connections**: `mysql_global_status_threads_connected`
- **Query Rate**: `rate(mysql_global_status_queries[5m])`
- **Slow Queries**: `mysql_global_status_slow_queries`

### 4. Kubernetes Cluster Metrics
- **Node CPU Usage**: `100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`
- **Node Memory Usage**: `(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100`
- **Pod Restarts**: `kube_pod_container_status_restarts_total`

## Alert Rules

### Critical Alerts:
1. **HighPodCPU**: Pod CPU > 80% for 5 minutes
2. **Nginx5xxErrors**: 5xx error rate > 0.1/s for 2 minutes
3. **PodDown**: Pod not running for 5 minutes

### Warning Alerts:
1. **HighMemoryUsage**: Memory > 85% for 10 minutes
2. **MySQLHighConnections**: Connections > 90% of max_connections
