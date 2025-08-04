# Kubernetes: From Zero to Production

A comprehensive step-by-step guide to Kubernetes operations, from basic pod management to advanced deployment strategies.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Basic Operations](#basic-operations)
- [Intermediate Operations](#intermediate-operations)
- [Advanced Operations](#advanced-operations)
- [Monitoring and Debugging](#monitoring-and-debugging)

## Prerequisites

### 1. Install Required Tools

```bash
# Install Docker
brew install --cask docker

# Install kubectl (Kubernetes CLI)
brew install kubectl

# Install minikube (local Kubernetes cluster)
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-darwin-arm64
sudo install minikube-darwin-arm64 /usr/local/bin/minikube

# Verify installations
docker --version
kubectl version --client
minikube version
```

### 2. Setup Local Environment

```bash
# Start minikube cluster
minikube start --kubernetes-version=v1.33.1

# Verify cluster is running
kubectl cluster-info

# Set alias for convenience
alias k=kubectl

# Start Docker Desktop (if not running)
open -a Docker
```

## Docker Application Development

### 1. Create a Simple Web Application

**Create project structure:**
```bash
mkdir my-app
cd my-app
```

**Create package.json:**
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.mjs"
  },
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

**Create index.mjs (web server):**
```javascript
import express from 'express'
import os from 'os'

const app = express()
const PORT = 3000

app.get("/", (req, res) => {
  const helloMessage = `<h1>Hello from ${os.hostname()}</h1>`
  console.log(helloMessage)
  res.send(helloMessage)
})

app.listen(PORT, () => {
  console.log(`Web server is listening at port ${PORT}`)
})
```

### 2. Create Dockerfile

```dockerfile
FROM node:alpine

WORKDIR /app

EXPOSE 3000

COPY package.json package-lock.json ./

RUN npm install

COPY . ./

CMD ["npm", "start"]
```

### 3. Build and Test Docker Image

```bash
# Install dependencies
npm install

# Build Docker image
docker build . -t your-username/app-name:latest

# List local images
docker images

# Run container locally
docker run -d -p 3000:3000 your-username/app-name:latest

# Check if container is running
docker ps

# Test the application
curl http://localhost:3000

# Stop container
docker stop <container-id>
```

### 4. Working with Containers

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs <container-id>

# Follow logs in real-time
docker logs -f <container-id>

# Execute commands inside container
docker exec -it <container-id> /bin/sh

# Copy files from/to container
docker cp <container-id>:/app/file.txt ./local-file.txt

# Inspect container details
docker inspect <container-id>
```

### 5. Push to Docker Hub

```bash
# Login to Docker Hub
docker login

# Tag image with your username
docker tag app-name:latest your-username/app-name:latest

# Push to Docker Hub
docker push your-username/app-name:latest

# Push multiple tags
docker build . -t your-username/app-name:latest -t your-username/app-name:1.0.0
docker push your-username/app-name --all-tags
```

## Basic Operations

### 1. Understanding Kubernetes Components

**Pod**: The smallest deployable unit in Kubernetes
**Deployment**: Manages a set of identical pods
**Service**: Exposes pods to network traffic
**Namespace**: Provides isolation between resources

### 2. Basic Pod Operations

```bash
# Create a simple pod
kubectl run nginx --image=nginx

# List all pods
kubectl get pods

# Get detailed information about a pod
kubectl describe pod <pod-name>

# Delete a pod
kubectl delete pod <pod-name>
```

### 3. Basic Deployment Operations

```bash
# Create a deployment
kubectl create deployment my-app --image=nginx

# List deployments
kubectl get deployments

# Scale deployment
kubectl scale deployment my-app --replicas=3

# Delete deployment
kubectl delete deployment my-app
```

## Intermediate Operations

### 1. Container Image Management (Advanced)

```bash
# Build Docker image
docker build . -t your-username/app-name:version

# Tag image with multiple versions
docker build . -t your-username/app-name:latest -t your-username/app-name:1.0.0

# Push to registry
docker login
docker push your-username/app-name --all-tags

# List local images
docker images

# Remove unused images
docker image prune

# Remove all unused images
docker image prune -a

# View image history
docker history your-username/app-name:latest

### 2. Service Creation and Management

```bash
# Expose deployment as service
kubectl expose deployment app-name --type=LoadBalancer --port=80 --target-port=3000

# List services
kubectl get services

# Get service details
kubectl describe service <service-name>

# Delete service
kubectl delete service <service-name>
```

### 3. Network Access

```bash
# Get minikube IP
minikube ip

# Access service via tunnel (for LoadBalancer type)
minikube tunnel

# Get service URL
minikube service <service-name> --url
```

## Advanced Operations

### 1. YAML Configuration Files

**deployment.yaml** - Defines deployment with resource limits:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-web-hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: k8s-web-hello
  template:
    metadata:
      labels:
        app: k8s-web-hello
    spec:
      containers:
      - name: k8s-web-hello
        image: your-username/app-name:version
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

**service.yaml** - Defines service configuration:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: k8s-web-hello
spec:
  type: LoadBalancer
  selector:
    app: k8s-web-hello
  ports:
  - port: 3333
    targetPort: 3000
```

### 2. Apply YAML Configurations

```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Apply service
kubectl apply -f service.yaml

# Apply multiple files
kubectl apply -f .
```

### 3. Rolling Updates and Rollbacks

```bash
# Update image version
kubectl set image deployment/app-name container-name=new-image:version

# Monitor rollout status
kubectl rollout status deployment/app-name

# Rollback to previous version
kubectl rollout undo deployment/app-name

# Check rollout history
kubectl rollout history deployment/app-name
```

### 4. Scaling and Resource Management

```bash
# Scale deployment
kubectl scale deployment app-name --replicas=5

# Auto-scale based on CPU usage
kubectl autoscale deployment app-name --min=2 --max=10 --cpu-percent=80

# View resource usage
kubectl top pods
kubectl top nodes
```

## Monitoring and Debugging

### 1. Kubernetes Dashboard

```bash
# Start minikube dashboard
minikube dashboard
```
**Dashboard**: Web-based UI for managing Kubernetes clusters. Provides visual interface for pods, services, deployments, and other resources.

### 2. Debugging Commands

```bash
# Get pod logs
kubectl logs <pod-name>

# Follow logs in real-time
kubectl logs -f <pod-name>

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/bash

# Access minikube VM
minikube ssh
```

### 3. Cluster Information

```bash
# Get cluster info
kubectl cluster-info

# Get nodes
kubectl get nodes

# Describe node
kubectl describe node <node-name>

# Get all resources
kubectl get all
```

## Best Practices

1. **Resource Management**: Always specify resource requests and limits
2. **Versioning**: Use semantic versioning for container images
3. **Rolling Updates**: Use rolling update strategy for zero-downtime deployments
4. **Monitoring**: Set up proper monitoring and logging
5. **Security**: Use namespaces for isolation and RBAC for access control

## Common Commands Reference

### Docker Commands
| Command | Description |
|---------|-------------|
| `docker build . -t name:tag` | Build Docker image |
| `docker run -d -p host:container name:tag` | Run container in background |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker logs <container>` | View container logs |
| `docker exec -it <container> /bin/sh` | Access container shell |
| `docker push username/image:tag` | Push image to registry |
| `docker pull username/image:tag` | Pull image from registry |
| `docker images` | List local images |
| `docker stop <container>` | Stop running container |

### Kubernetes Commands
| Command | Description |
|---------|-------------|
| `kubectl get pods` | List all pods |
| `kubectl describe pod <name>` | Get detailed pod info |
| `kubectl logs <pod>` | Get pod logs |
| `kubectl exec -it <pod> -- /bin/bash` | Access pod shell |
| `kubectl apply -f file.yaml` | Apply YAML configuration |
| `kubectl delete -f file.yaml` | Delete resources from YAML |
| `kubectl scale deployment <name> --replicas=N` | Scale deployment |
| `kubectl rollout status deployment/<name>` | Check rollout status |
| `minikube tunnel` | Enable LoadBalancer access |
| `minikube dashboard` | Open web dashboard |

## Troubleshooting

### Common Issues

#### Docker Issues
1. **Image build fails**: Check Dockerfile syntax and dependencies
2. **Container won't start**: Check port conflicts and resource limits
3. **Push to registry fails**: Verify Docker Hub credentials
4. **Container exits immediately**: Check CMD/ENTRYPOINT and application logs

#### Kubernetes Issues
1. **Pod stuck in Pending**: Check resource availability and node capacity
2. **Service not accessible**: Verify service type and port configuration
3. **Image pull errors**: Check image name and registry authentication
4. **Resource limits exceeded**: Monitor resource usage and adjust limits

### Debugging Steps

#### Docker Debugging
1. Check container status: `docker ps -a`
2. View container logs: `docker logs <container>`
3. Inspect container: `docker inspect <container>`
4. Test container: `docker exec -it <container> /bin/sh`
5. Check image details: `docker history <image>`

#### Kubernetes Debugging
1. Check pod status: `kubectl get pods`
2. View pod events: `kubectl describe pod <name>`
3. Check logs: `kubectl logs <pod>`
4. Verify service: `kubectl describe service <name>`
5. Test connectivity: `kubectl exec -it <pod> -- curl localhost:port`
