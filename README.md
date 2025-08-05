# Kubernetes: From Zero to Production

A comprehensive step-by-step guide to Kubernetes operations, from basic pod management to advanced deployment strategies.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Docker Application Development](#docker-application-development)
- [Basic Operations](#basic-operations)
- [Advanced Operations](#advanced-operations)
- [Multi-Deployment Architecture](#multi-deployment-architecture)
- [Monitoring and Debugging](#monitoring-and-debugging)
- [Common Commands Reference](#common-commands-reference)

## Prerequisites

### Install Required Tools

```bash
# Install Docker, kubectl, and minikube
brew install --cask docker
brew install kubectl
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-darwin-arm64
sudo install minikube-darwin-arm64 /usr/local/bin/minikube

# Setup environment
minikube start --kubernetes-version=v1.33.1
kubectl cluster-info
alias k=kubectl
open -a Docker
```

## Docker Application Development

### Create and Deploy Application

**1. Create simple web app:**
```bash
mkdir my-app && cd my-app
npm init -y
npm install express
```

**2. Create index.mjs:**
```javascript
import express from 'express'
const app = express()
app.get("/", (req, res) => res.send(`<h1>Hello from ${os.hostname()}</h1>`))
app.listen(3000, () => console.log('Server running on port 3000'))
```

**3. Create Dockerfile:**
```dockerfile
FROM node:alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

**4. Build and deploy:**
```bash
docker build . -t your-username/app:latest
docker run -d -p 3000:3000 your-username/app:latest
docker login && docker push your-username/app:latest
```

### Container Management

```bash
# Basic container operations
docker ps                    # List running containers
docker logs <container>      # View logs
docker exec -it <container> /bin/sh  # Access shell
docker stop <container>      # Stop container
```

## Basic Operations

### Kubernetes Components

**Pod**: Smallest deployable unit
**Deployment**: Manages identical pods
**Service**: Exposes pods to network
**Namespace**: Provides isolation

### Basic Commands

```bash
# Pod operations
kubectl run nginx --image=nginx
kubectl get pods
kubectl describe pod <name>
kubectl delete pod <name>

# Deployment operations
kubectl create deployment my-app --image=nginx
kubectl get deployments
kubectl scale deployment my-app --replicas=3
kubectl delete deployment my-app

# Service operations
kubectl expose deployment app-name --type=LoadBalancer --port=80 --target-port=3000
kubectl get services
kubectl describe service <name>
```

## Advanced Operations

### YAML Configuration

**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: your-username/app:latest
        resources:
          requests:
            memory: "64Mi"
            cpu: "125m"
          limits:
            memory: "128Mi"
            cpu: "250m"
        ports:
        - containerPort: 3000
```

**service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 3000
```

### Apply and Manage

```bash
# Apply configurations
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f .

# Rolling updates
kubectl set image deployment/my-app my-app=new-image:version
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app

# Scaling
kubectl scale deployment my-app --replicas=5
kubectl autoscale deployment my-app --min=2 --max=10 --cpu-percent=80
```

## Multi-Deployment Architecture

### Deploy Multiple Applications

```bash
# Apply multiple deployments
kubectl apply -f app1.yaml -f app2.yaml

# Monitor all resources
kubectl get all
kubectl get deployments
kubectl get services
kubectl get pods -o wide
```

### Service Communication

```bash
# Test inter-service communication
kubectl exec -it <pod> -- curl service-name:port

# Port forwarding
kubectl port-forward service/<name> 8080:80

# External access (minikube)
minikube tunnel
curl $(minikube service <name> --url)
```

### Example Multi-Tier Setup

**Frontend (app.yaml):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: your-username/frontend:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
```

**Backend (api.yaml):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: your-username/backend:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
```

## Monitoring and Debugging

### Dashboard and Monitoring

```bash
# Start dashboard
minikube dashboard

# Debug commands
kubectl logs <pod>
kubectl logs -f <pod>
kubectl exec -it <pod> -- /bin/bash
kubectl describe pod <name>
kubectl describe service <name>
```

### Cluster Information

```bash
kubectl cluster-info
kubectl get nodes
kubectl get endpoints
kubectl top pods
kubectl top nodes
```

## Common Commands Reference

### Docker Commands
| Command | Description |
|---------|-------------|
| `docker build . -t name:tag` | Build image |
| `docker run -d -p host:container name:tag` | Run container |
| `docker ps` | List containers |
| `docker logs <container>` | View logs |
| `docker exec -it <container> /bin/sh` | Access shell |
| `docker push username/image:tag` | Push to registry |

### Kubernetes Commands
| Command | Description |
|---------|-------------|
| `kubectl get pods` | List pods |
| `kubectl get all` | List all resources |
| `kubectl apply -f file.yaml` | Apply configuration |
| `kubectl describe <resource> <name>` | Get details |
| `kubectl logs <pod>` | View logs |
| `kubectl exec -it <pod> -- /bin/bash` | Access shell |
| `kubectl scale deployment <name> --replicas=N` | Scale |
| `kubectl rollout status deployment/<name>` | Check rollout |
| `minikube tunnel` | Enable LoadBalancer |
| `minikube dashboard` | Open dashboard |

## Troubleshooting

### Common Issues

**Docker:**
- Image build fails: Check Dockerfile syntax
- Container won't start: Check port conflicts
- Push fails: Verify Docker Hub credentials

**Kubernetes:**
- Pod stuck in Pending: Check resource availability
- Service not accessible: Verify port configuration
- Image pull errors: Check registry authentication

### Debugging Steps

1. Check status: `kubectl get pods`, `docker ps -a`
2. View logs: `kubectl logs <pod>`, `docker logs <container>`
3. Inspect details: `kubectl describe <resource>`, `docker inspect <container>`
4. Test connectivity: `kubectl exec -it <pod> -- curl localhost:port`
