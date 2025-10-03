# Kubernetes (K8s) - Container Orchestration Platform

## Overview

Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It provides a container-centric management environment and coordinates with Docker, rkt, and other container runtimes.

### Key Features
- **Automated Scheduling**: Automatically places containers based on resource requirements
- **Self-Healing**: Restarts failed containers and reschedules them
- **Horizontal Scaling**: Automatically scales applications based on demand
- **Service Discovery**: Groups containers and provides load balancing
- **Automated Rollouts**: Gradually rolls out changes while monitoring application health
- **Configuration Management**: Manages application configuration consistently

## Core Concepts

### Cluster Architecture
- **Master Node**: Controls the cluster (API Server, Scheduler, Controller Manager, etcd)
- **Worker Nodes**: Run applications (Kubelet, Kube-proxy, Container Runtime)
- **etcd**: Distributed key-value store for cluster data
- **Pods**: Smallest deployable units in Kubernetes

### Key Components

#### Master Components
- **API Server**: Exposes Kubernetes API
- **Scheduler**: Assigns pods to nodes
- **Controller Manager**: Runs controller processes
- **etcd**: Consistent and highly-available key-value store

#### Node Components
- **Kubelet**: Agent that runs on each node
- **Kube-proxy**: Network proxy and load balancer
- **Container Runtime**: Software that runs containers (Docker, containerd, etc.)

### Kubernetes Objects
- **Pod**: Group of one or more containers
- **Service**: Network abstraction for pods
- **Deployment**: Manages replica sets and pods
- **StatefulSet**: Manages stateful applications
- **DaemonSet**: Ensures pods run on all nodes
- **Job**: Runs one or more pods to completion
- **CronJob**: Runs jobs on a schedule
- **ConfigMap**: Stores configuration data
- **Secret**: Stores sensitive data
- **Ingress**: Manages external access to services
- **PersistentVolume**: Storage resource
- **PersistentVolumeClaim**: Request for storage

## Installation & Setup

### Prerequisites
- **Operating System**: Linux (Ubuntu, CentOS, etc.)
- **Memory**: Minimum 2GB RAM per node
- **CPU**: Minimum 2 CPU cores
- **Network**: Proper network connectivity between nodes
- **Container Runtime**: Docker, containerd, or CRI-O

### Installation Methods

#### Using kubeadm (Recommended)
```bash
# Install Docker
sudo apt update
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Install kubeadm, kubelet, kubectl
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Initialize cluster (on master node)
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Set up kubectl (for regular user)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install network plugin (e.g., Flannel)
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# Join worker nodes (run on worker nodes)
sudo kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash <hash>
```

#### Using Minikube (Local Development)
```bash
# Install Minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube /usr/local/bin/

# Start Minikube cluster
minikube start

# Check cluster status
minikube status

# Access Kubernetes dashboard
minikube dashboard
```

#### Using k3s (Lightweight Kubernetes)
```bash
# Install k3s
curl -sfL https://get.k3s.io | sh -

# Check status
sudo systemctl status k3s

# Get kubeconfig
sudo cat /etc/rancher/k3s/k3s.yaml
```

## Core Kubernetes Resources

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    env:
    - name: ENV_VAR
      value: "production"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://localhost:5432/mydb"
  debug_mode: "true"
  max_connections: "100"
```

### Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded "admin"
  password: MWYyZDFlMmU2N2Rm  # base64 encoded password
```

### PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
```

### PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: manual
```

## Kubernetes Commands

### Cluster Management
```bash
# Get cluster information
kubectl cluster-info

# Get nodes
kubectl get nodes

# Get nodes with more details
kubectl get nodes -o wide

# Describe node
kubectl describe node <node-name>

# Get cluster events
kubectl get events
```

### Pod Management
```bash
# Get pods
kubectl get pods

# Get pods in all namespaces
kubectl get pods --all-namespaces

# Get pods with more details
kubectl get pods -o wide

# Describe pod
kubectl describe pod <pod-name>

# Create pod from file
kubectl create -f pod.yaml

# Apply configuration
kubectl apply -f pod.yaml

# Delete pod
kubectl delete pod <pod-name>

# Get pod logs
kubectl logs <pod-name>

# Follow pod logs
kubectl logs -f <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash

# Port forward to pod
kubectl port-forward <pod-name> 8080:80
```

### Deployment Management
```bash
# Get deployments
kubectl get deployments

# Create deployment
kubectl create deployment nginx --image=nginx

# Scale deployment
kubectl scale deployment nginx --replicas=3

# Update deployment image
kubectl set image deployment/nginx nginx=nginx:1.22

# Rollback deployment
kubectl rollout undo deployment/nginx

# Check rollout status
kubectl rollout status deployment/nginx

# Get rollout history
kubectl rollout history deployment/nginx
```

### Service Management
```bash
# Get services
kubectl get services

# Create service
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP

# Delete service
kubectl delete service <service-name>
```

### Configuration Management
```bash
# Get configmaps
kubectl get configmaps

# Create configmap from file
kubectl create configmap app-config --from-file=config.properties

# Get secrets
kubectl get secrets

# Create secret from literal
kubectl create secret generic db-secret --from-literal=password=mypassword
```

## Kubernetes Networking

### Network Models
- **ClusterIP**: Exposes service on cluster-internal IP
- **NodePort**: Exposes service on each node's IP at static port
- **LoadBalancer**: Exposes service externally using cloud provider's load balancer
- **ExternalName**: Maps service to external name using CNAME

### Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

### Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-traffic
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 80
```

## Kubernetes Security

### RBAC (Role-Based Access Control)
```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Security Context
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: secure-container
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      capabilities:
        drop:
        - ALL
```

## Kubernetes Monitoring

### Metrics Server
```bash
# Install Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Check node resource usage
kubectl top node

# Check pod resource usage
kubectl top pod
```

### Logging
```bash
# View pod logs
kubectl logs <pod-name>

# View logs from all containers in pod
kubectl logs <pod-name> --all-containers=true

# View logs from previous container instance
kubectl logs <pod-name> --previous

# View logs with timestamp
kubectl logs <pod-name> --timestamps=true
```

## Kubernetes Helm

### Helm Chart Structure
```
mychart/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
└── README.md
```

### Helm Commands
```bash
# Create a chart
helm create mychart

# Install a chart
helm install my-release ./mychart

# List releases
helm list

# Upgrade a release
helm upgrade my-release ./mychart

# Rollback a release
helm rollback my-release 1

# Uninstall a release
helm uninstall my-release
```

## Interview Questions

### Beginner Level
1. **What is Kubernetes?**
   - Kubernetes is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications

2. **What are the main components of Kubernetes?**
   - Master components (API Server, Scheduler, Controller Manager, etcd) and Node components (Kubelet, Kube-proxy, Container Runtime)

3. **What is a Pod?**
   - A Pod is the smallest deployable unit in Kubernetes, containing one or more containers

4. **What is the difference between Deployment and StatefulSet?**
   - Deployment manages stateless applications, while StatefulSet manages stateful applications with persistent storage

### Intermediate Level
1. **What is the difference between a Service and an Ingress?**
   - Service provides internal load balancing and discovery, while Ingress manages external access to services

2. **What is a ConfigMap and how is it used?**
   - ConfigMap stores configuration data as key-value pairs and can be mounted as volumes or used as environment variables

3. **What is the purpose of a PersistentVolume?**
   - PersistentVolume provides persistent storage that survives pod restarts and rescheduling

4. **How does Kubernetes handle scaling?**
   - Kubernetes can scale horizontally by increasing/decreasing the number of pod replicas and vertically by adjusting resource limits

### Advanced Level
1. **What is Kubernetes RBAC and how does it work?**
   - RBAC (Role-Based Access Control) manages access to Kubernetes resources through Roles, RoleBindings, ClusterRoles, and ClusterRoleBindings

2. **What is the difference between kubeadm, kops, and kubespray?**
   - kubeadm is for bootstrapping clusters, kops is for deploying production-grade clusters on AWS, kubespray is for deploying clusters on multiple platforms

3. **How does Kubernetes handle network policies?**
   - Network policies define rules for pod communication, controlling ingress and egress traffic

4. **What is Kubernetes operator pattern?**
   - Operators are custom controllers that extend Kubernetes functionality to manage complex applications

## Best Practices

### Resource Management
- **Resource Requests**: Always specify resource requests for containers
- **Resource Limits**: Set appropriate limits to prevent resource starvation
- **Health Checks**: Implement liveness and readiness probes
- **Pod Disruption Budgets**: Ensure application availability during maintenance

### Security
- **Least Privilege**: Use RBAC with minimal required permissions
- **Network Policies**: Implement network policies to restrict traffic
- **Secrets Management**: Use Secrets for sensitive data, not ConfigMaps
- **Image Security**: Use trusted images and scan for vulnerabilities

### Monitoring & Logging
- **Metrics Collection**: Implement metrics collection and monitoring
- **Log Aggregation**: Centralize logs from all pods
- **Alerting**: Set up alerts for critical events
- **Health Monitoring**: Monitor cluster and application health

### Deployment Strategies
- **Rolling Updates**: Use rolling updates for zero-downtime deployments
- **Blue-Green Deployment**: Deploy to parallel environment and switch traffic
- **Canary Deployment**: Roll out changes to subset of users first
- **A/B Testing**: Test different versions simultaneously

## Troubleshooting

### Common Issues
```bash
# Check pod status
kubectl get pods

# Check pod events
kubectl describe pod <pod-name>

# Check pod logs
kubectl logs <pod-name>

# Check node status
kubectl get nodes

# Check cluster events
kubectl get events --all-namespaces

# Check service endpoints
kubectl get endpoints

# Check network connectivity
kubectl exec -it <pod-name> -- ping <service-name>
```

### Debugging Tools
```bash
# Use busybox for debugging
kubectl run -it --rm debug --image=busybox -- sh

# Check DNS resolution
kubectl exec -it <pod-name> -- nslookup kubernetes.default

# Check service connectivity
kubectl exec -it <pod-name> -- wget -qO- <service-name>

# Check resource usage
kubectl top nodes
kubectl top pods
```

## Resources

### Official Documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/)
- [Kubernetes Tasks](https://kubernetes.io/docs/tasks/)

### Learning Resources
- [Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [Kubernetes Training](https://kubernetes.io/training/)
- [Interactive Tutorials](https://kubernetes.io/docs/tutorials/)

### Community
- [Kubernetes Community](https://kubernetes.io/community/)
- [Kubernetes Slack](https://slack.k8s.io/)
- [Kubernetes GitHub](https://github.com/kubernetes/kubernetes)