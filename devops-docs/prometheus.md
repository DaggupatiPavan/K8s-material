# Prometheus - Monitoring and Alerting System

## Overview

Prometheus is an open-source monitoring and alerting system originally built at SoundCloud. Since its inception in 2012, many companies and organizations have adopted Prometheus, and the project has a very active developer and user community. It is now a standalone open source project and maintained independently of any company.

### Key Features
- **Multi-dimensional Data Model**: Time series data identified by metric name and key/value pairs
- **PromQL**: Flexible query language to leverage this dimensionality
- **Time Series Collection**: Pull model over HTTP for time series collection
- **Push Gateway**: Support for push-based metrics collection
- **Service Discovery**: Automatic discovery of targets
- **Alerting**: Built-in alerting with Alertmanager
- **Visualization**: Integration with Grafana and other UI tools
- **Client Libraries**: Support for multiple programming languages

## Core Concepts

### Prometheus Architecture
- **Prometheus Server**: Main component that scrapes and stores time series data
- **Client Libraries**: Instrument applications to expose metrics
- **Push Gateway**: Support for short-lived jobs
- **Alertmanager**: Handle alerts sent by Prometheus server
- **Exporters**: Export metrics from third-party systems
- **Service Discovery**: Automatically discover monitoring targets
- **Visualization**: Tools like Grafana for data visualization

### Key Components
- **Metrics**: Quantitative measurements of a system
- **Labels**: Key/value pairs that distinguish metrics
- **Time Series**: Stream of timestamped values belonging to the same metric
- **Instances**: Individual targets that can be scraped
- **Jobs**: Collection of instances with the same purpose
- **Operators**: Functions that can be applied to time series
- **Recording Rules**: Pre-compute frequently needed expressions
- **Alerting Rules**: Define alert conditions

### Data Model
- **Metric Name**: Name of the metric (e.g., http_requests_total)
- **Labels**: Key/value pairs that distinguish metrics (e.g., method="GET", status="200")
- **Metric Value**: Numeric value of the metric
- **Timestamp**: When the value was recorded

## Installation & Setup

### Prerequisites
- **Operating System**: Linux, macOS, Windows
- **Memory**: Minimum 2GB RAM (4GB recommended)
- **Disk**: Minimum 10GB free space (SSD recommended)
- **Network**: Port 9090 for web interface

### Installation Methods

#### Using Binary Download
```bash
# Download Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz

# Extract Prometheus
tar xzf prometheus-2.37.0.linux-amd64.tar.gz
cd prometheus-2.37.0.linux-amd64

# Run Prometheus
./prometheus --config.file=prometheus.yml

# Access web interface
# http://localhost:9090
```

#### Using Package Manager (Ubuntu/Debian)
```bash
# Add Prometheus repository
sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common
wget -q -O - https://packagecloud.io/gpg.key | sudo apt-key add -
echo "deb https://packagecloud.io/prometheus-rpm/el/7/ $main" | sudo tee /etc/apt/sources.list.d/prometheus.list

# Install Prometheus
sudo apt-get update
sudo apt-get install prometheus

# Start Prometheus
sudo systemctl start prometheus
sudo systemctl enable prometheus

# Check status
sudo systemctl status prometheus
```

#### Using Docker
```bash
# Run Prometheus in Docker
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest

# Run with custom configuration
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v $(pwd)/rules:/etc/prometheus/rules \
  prom/prometheus:latest \
  --config.file=/etc/prometheus/prometheus.yml \
  --web.enable-lifecycle
```

## Prometheus Configuration

### Basic Configuration (prometheus.yml)
```yaml
# Global configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

# Rule files
rule_files:
  - "rules/*.yml"

# Scrape configuration
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  # Application metrics
  - job_name: 'my_app'
    static_configs:
      - targets: ['app-server:8080']

  # Kubernetes pods
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### Advanced Configuration
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'codelab-monitor'

# Rule files
rule_files:
  - "first_rules.yml"
  - "second_rules.yml"

# Scrape configuration
scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    scrape_interval: 5s
    static_configs:
      - targets: ['node1:9100', 'node2:9100', 'node3:9100']

  - job_name: 'docker'
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
        filters:
          - name: label
            values: [com.docker.compose.service]

  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
    scheme: http
    timeout: 10s

# Remote write configuration
remote_write:
  - url: "http://remote-storage:9090/api/v1/write"
    queue_config:
      max_shards: 100
      min_shards: 10
      max_samples_per_send: 100
      batch_send_deadline: 5s
      retry_on_http_429: true

# Remote read configuration
remote_read:
  - url: "http://remote-storage:9090/api/v1/read"
    read_recent: true
```

## Prometheus Metrics

### Metric Types

#### Counter
```python
from prometheus_client import Counter

# Create a counter
http_requests_total = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint'])

# Increment counter
http_requests_total.labels(method='GET', endpoint='/api').inc()
http_requests_total.labels(method='POST', endpoint='/api').inc()
```

#### Gauge
```python
from prometheus_client import Gauge

# Create a gauge
memory_usage = Gauge('memory_usage_bytes', 'Memory usage in bytes')

# Set gauge value
memory_usage.set(1024 * 1024 * 100)  # 100MB

# Increment/decrement gauge
memory_usage.inc(1024 * 1024)  # Add 1MB
memory_usage.dec(1024 * 1024)  # Subtract 1MB
```

#### Histogram
```python
from prometheus_client import Histogram

# Create a histogram
request_duration = Histogram('request_duration_seconds', 'Request duration in seconds')

# Observe value
request_duration.observe(0.5)  # 500ms

# Use as decorator
@request_duration.time()
def process_request():
    # Process request
    pass
```

#### Summary
```python
from prometheus_client import Summary

# Create a summary
response_size = Summary('response_size_bytes', 'Response size in bytes')

# Observe value
response_size.observe(1024)  # 1KB

# Use as decorator
@response_size.time()
def process_response():
    # Process response
    pass
```

### Metric Labels
```python
from prometheus_client import Counter

# Create counter with labels
http_requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Use labels
http_requests_total.labels(method='GET', endpoint='/api', status='200').inc()
http_requests_total.labels(method='POST', endpoint='/api', status='400').inc()
```

## PromQL (Prometheus Query Language)

### Basic Queries

#### Select all time series for a metric
```
http_requests_total
```

#### Select time series with specific labels
```
http_requests_total{method="GET"}
http_requests_total{method="GET", endpoint="/api"}
```

#### Select time series with label matching
```
http_requests_total{method=~"GET|POST"}
http_requests_total{endpoint!="/health"}
```

### Operators

#### Arithmetic Operators
```
# Addition
http_requests_total + 100

# Subtraction
http_requests_total - 50

# Multiplication
http_requests_total * 2

# Division
http_requests_total / 60

# Modulo
http_requests_total % 1000
```

#### Comparison Operators
```
# Equality
http_requests_total == 100

# Not equal
http_requests_total != 0

# Greater than
http_requests_total > 100

# Less than
http_requests_total < 1000

# Greater than or equal
http_requests_total >= 50

# Less than or equal
http_requests_total <= 500
```

#### Logical Operators
```
# AND
http_requests_total > 100 and memory_usage > 80

# OR
http_requests_total > 100 or memory_usage > 80

# Unless
http_requests_total > 100 unless memory_usage > 80
```

### Functions

#### Rate Functions
```
# Rate of increase over time
rate(http_requests_total[5m])

# Increase over time
increase(http_requests_total[1h])

# Per-second rate of increase
irate(http_requests_total[5m])
```

#### Aggregation Functions
```
# Sum
sum(http_requests_total)

# Average
avg(http_requests_total)

# Maximum
max(http_requests_total)

# Minimum
min(http_requests_total)

# Count
count(http_requests_total)

# Standard deviation
stddev(http_requests_total)

# Standard variance
stdvar(http_requests_total)
```

#### Time Functions
```
# Time series over time
rate(cpu_usage[5m])

# Change over time
delta(cpu_usage[1h])

# Predict future values
predict_linear(cpu_usage[1h], 3600)
```

### Advanced Queries

#### Calculate error rate
```
# Error rate as percentage
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100
```

#### Calculate 95th percentile
```
# 95th percentile of response time
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

#### Calculate availability
```
# Availability percentage
rate(up{job="my_app"}[5m]) * 100
```

## Recording Rules

### Recording Rule Configuration
```yaml
# rules/first_rules.yml
groups:
  - name: example
    interval: 30s
    rules:
    - record: job:http_inprogress_requests:sum
      expr: sum(http_inprogress_requests) by (job)
    
    - record: job:http_requests_total:sum
      expr: sum(http_requests_total) by (job)
    
    - record: job:http_requests_total:rate5m
      expr: sum(rate(http_requests_total[5m])) by (job)
    
    - record: job:http_request_duration_seconds:mean5m
      expr: sum(rate(http_request_duration_seconds_sum[5m])) by (job) / 
             sum(rate(http_request_duration_seconds_count[5m])) by (job)
```

### Complex Recording Rules
```yaml
# rules/complex_rules.yml
groups:
  - name: system
    interval: 1m
    rules:
    - record: node_cpu_utilization:avg5m
      expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    
    - record: node_memory_utilization:avg5m
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
    
    - record: node_disk_utilization:avg5m
      expr: (node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_avail_bytes{mountpoint="/"}) / 
             node_filesystem_size_bytes{mountpoint="/"} * 100
```

## Alerting Rules

### Alerting Rule Configuration
```yaml
# rules/alert_rules.yml
groups:
  - name: example
    rules:
    - alert: HighRequestLatency
      expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
      for: 10m
      labels:
        severity: page
      annotations:
        summary: High request latency on {{ $labels.instance }}
        description: "{{ $labels.instance }} has a high request latency of {{ $value }} seconds."

    - alert: HighMemoryUsage
      expr: node_memory_utilization:avg5m > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High memory usage on {{ $labels.instance }}
        description: "{{ $labels.instance }} has memory usage above 80%."

    - alert: ServiceDown
      expr: up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: Service {{ $labels.instance }} is down
        description: "{{ $labels.instance }} has been down for more than 1 minute."
```

### Advanced Alerting Rules
```yaml
# rules/advanced_alerts.yml
groups:
  - name: system
    rules:
    - alert: HighCPUUsage
      expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: High CPU usage on {{ $labels.instance }}
        description: "{{ $labels.instance }} CPU usage is {{ $value }}% for 10 minutes."

    - alert: DiskSpaceLow
      expr: (1 - (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"})) * 100 > 85
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Low disk space on {{ $labels.instance }}
        description: "{{ $labels.instance }} disk space is {{ $value }}% full."

    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100 > 5
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: High error rate on {{ $labels.job }}
        description: "{{ $labels.job }} error rate is {{ $value }}% for 5 minutes."
```

## Exporters

### Node Exporter
```bash
# Install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xzf node_exporter-1.3.1.linux-amd64.tar.gz
cd node_exporter-1.3.1.linux-amd64

# Run Node Exporter
./node_exporter

# Access metrics
# http://localhost:9100/metrics
```

### Node Exporter Configuration
```yaml
# Add to prometheus.yml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
    scrape_interval: 15s
    metrics_path: /metrics
```

### Blackbox Exporter
```bash
# Install Blackbox Exporter
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.21.0/blackbox_exporter-0.21.0.linux-amd64.tar.gz
tar xzf blackbox_exporter-0.21.0.linux-amd64.tar.gz
cd blackbox_exporter-0.21.0.linux-amd64

# Configure blackbox.yml
cat > blackbox.yml << EOF
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: []
      method: GET
  icmp:
    prober: icmp
    timeout: 5s
  tcp_connect:
    prober: tcp
    timeout: 5s
EOF

# Run Blackbox Exporter
./blackbox_exporter --config.file=blackbox.yml
```

### Blackbox Exporter Configuration
```yaml
# Add to prometheus.yml
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://www.google.com
        - https://www.github.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

### MySQL Exporter
```bash
# Install MySQL Exporter
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.14.0/mysqld_exporter-0.14.0.linux-amd64.tar.gz
tar xzf mysqld_exporter-0.14.0.linux-amd64.tar.gz
cd mysqld_exporter-0.14.0.linux-amd64

# Create MySQL user
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'password' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';

# Create .my.cnf
cat > .my.cnf << EOF
[client]
user=exporter
password=password
EOF

# Run MySQL Exporter
./mysqld_exporter --config.my-cnf=.my.cnf
```

## Alertmanager

### Alertmanager Installation
```bash
# Download Alertmanager
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
tar xzf alertmanager-0.24.0.linux-amd64.tar.gz
cd alertmanager-0.24.0.linux-amd64

# Run Alertmanager
./alertmanager --config.file=alertmanager.yml

# Access web interface
# http://localhost:9093
```

### Alertmanager Configuration
```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
  routes:
  - match:
      service: mysql
    receiver: 'mysql-team'

receivers:
- name: 'web.hook'
  email_configs:
    - to: 'devops@example.com'
      subject: 'Prometheus Alert: {{ .GroupLabels.alertname }}'
      body: |
        {{ range .Alerts }}
        Alert: {{ .Annotations.summary }}
        Description: {{ .Annotations.description }}
        Labels: {{ .Labels }}
        {{ end }}
  webhook_configs:
    - url: 'http://127.0.0.1:5001/webhook'

- name: 'mysql-team'
  email_configs:
    - to: 'mysql-team@example.com'
      subject: 'MySQL Alert: {{ .GroupLabels.alertname }}'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

## Grafana Integration

### Grafana Installation
```bash
# Install Grafana (Ubuntu/Debian)
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt-get update
sudo apt-get install grafana

sudo systemctl start grafana
sudo systemctl enable grafana
```

### Add Prometheus Data Source
```json
{
  "type": "prometheus",
  "url": "http://localhost:9090",
  "access": "proxy",
  "basicAuth": false,
  "isDefault": true
}
```

### Example Dashboard Queries
```
# CPU Usage
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk Usage
(1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

# HTTP Requests Rate
rate(http_requests_total[5m])

# HTTP Error Rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100
```

## Interview Questions

### Beginner Level
1. **What is Prometheus?**
   - Prometheus is an open-source monitoring and alerting system designed for reliability and scalability

2. **What are the main components of Prometheus?**
   - Prometheus Server, Client Libraries, Push Gateway, Alertmanager, Exporters, and Service Discovery

3. **What is PromQL?**
   - PromQL is the query language used to select and aggregate time series data in Prometheus

4. **What are the different types of metrics in Prometheus?**
   - Counter, Gauge, Histogram, and Summary

### Intermediate Level
1. **What is the difference between Counter and Gauge?**
   - Counter is a metric that only increases, while Gauge can go up and down

2. **How does Prometheus collect metrics?**
   - Prometheus uses a pull model where it scrapes metrics endpoints of targets

3. **What is the purpose of Alertmanager?**
   - Alertmanager handles alerts sent by Prometheus server and manages notifications

4. **What are exporters in Prometheus?**
   - Exporters are programs that expose metrics from third-party systems in Prometheus format

### Advanced Level
1. **How does Prometheus handle high cardinality metrics?**
   - Prometheus handles high cardinality through efficient storage and query optimization, but it's better to avoid high cardinality labels

2. **What is the difference between rate() and irate()?**
   - rate() calculates per-second average rate over time window, while irate() calculates per-second instant rate

3. **How do you optimize Prometheus performance?**
   - Use appropriate recording rules, optimize queries, use federation, and configure proper retention policies

4. **What is Prometheus federation?**
   - Federation allows a Prometheus server to scrape selected time series from another Prometheus server

## Best Practices

### Metric Design
- **Use Consistent Naming**: Follow consistent naming conventions for metrics
- **Limit Label Cardinality**: Avoid high cardinality labels
- **Use Appropriate Metric Types**: Choose the right metric type for your use case
- **Document Metrics**: Provide documentation for all custom metrics
- **Avoid PII**: Don't include personally identifiable information in metrics

### Configuration
- **Use Configuration Management**: Store Prometheus configuration in version control
- **Implement Recording Rules**: Pre-compute frequently needed expressions
- **Use Service Discovery**: Automate target discovery
- **Configure Proper Retention**: Set appropriate data retention policies
- **Monitor Prometheus**: Monitor the Prometheus server itself

### Alerting
- **Create Meaningful Alerts**: Create alerts that require action
- **Use Appropriate Severity**: Use appropriate severity levels for alerts
- **Set Proper Thresholds**: Set reasonable alert thresholds
- **Test Alert Rules**: Test alert rules before deploying
- **Review Alerts Regularly**: Regularly review and update alert rules

## Troubleshooting

### Common Issues
```bash
# Check Prometheus status
sudo systemctl status prometheus

# Check Prometheus logs
sudo journalctl -u prometheus -f

# Validate configuration
./promtool check config prometheus.yml

# Validate rules
./promtool check rules rules/*.yml

# Check metrics endpoint
curl http://localhost:9090/metrics

# Check query execution
curl 'http://localhost:9090/api/v1/query?query=up'
```

### Debugging Tips
- **Check Logs**: Always check Prometheus logs for error messages
- **Validate Configuration**: Use promtool to validate configuration
- **Test Queries**: Test queries in the Prometheus web interface
- **Monitor Resources**: Monitor Prometheus server resources
- **Check Network Connectivity**: Ensure network connectivity to targets

## Resources

### Official Documentation
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Prometheus Query Language](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Alertmanager Documentation](https://prometheus.io/docs/alerting/latest/alertmanager/)

### Learning Resources
- [Prometheus Tutorials](https://prometheus.io/docs/introduction/overview/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/naming/)
- [Prometheus Client Libraries](https://prometheus.io/docs/instrumenting/clientlibs/)

### Community
- [Prometheus GitHub](https://github.com/prometheus/prometheus)
- [Prometheus Community](https://prometheus.io/community/)
- [Prometheus Stack Overflow](https://stackoverflow.com/questions/tagged/prometheus)