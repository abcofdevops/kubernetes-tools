# Kubernetes Test Go Application

A lightweight Go web application designed for testing Kubernetes installations, ingress controllers, and various Kubernetes features. This app provides a simple HTTP server that returns the pod hostname, making it perfect for verifying cluster functionality and load balancing.

## Overview

This application serves as a minimal testing tool for Kubernetes environments. It's built with Go and containerized using Docker, following best practices with multi-stage builds and distroless base images for security and minimal footprint.

### Key Features

- **Lightweight**: Minimal Go HTTP server with distroless container image
- **Pod Identification**: Returns hostname/pod name for easy identification in multi-replica deployments
- **Kubernetes Ready**: Designed specifically for testing K8s deployments, services, and ingress
- **Security Focused**: Uses distroless base image to minimize attack surface

## Quick Start

### Prerequisites

- Docker installed
- Kubernetes cluster (minikube, kind, or any K8s cluster)
- kubectl configured to access your cluster

### Building the Docker Image

```bash
# Clone the repository
git clone https://github.com/abcofdevops/kubernetes-tools.git
cd kubernetes-tools/k8s-setup-test-go-app

# Build the Docker image
docker build -t k8s-test-go-app:latest .

# Run locally (optional)
docker run -p 8080:8080 k8s-test-go-app:latest
```

## Application Details

### Code Structure

- **main.go**: Simple HTTP server that responds with pod hostname
- **go.mod**: Go module definition (Go 1.24.6)
- **Dockerfile**: Multi-stage build with distroless final image

### HTTP Endpoints

- `GET /` - Returns "Hello from Pod: {hostname}"
- Server runs on port 8080

### Docker Image Details

The Dockerfile uses a two-stage build process:
1. **Build Stage**: Uses `golang:1.24` to compile the Go application
2. **Runtime Stage**: Uses `gcr.io/distroless/base:latest` for minimal, secure runtime

## Kubernetes Deployment Examples

### Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-test-go-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: k8s-test-go-app
  template:
    metadata:
      labels:
        app: k8s-test-go-app
    spec:
      containers:
      - name: k8s-test-go-app
        image: k8s-test-go-app:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: k8s-test-go-app-service
spec:
  selector:
    app: k8s-test-go-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-test-go-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: test-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: k8s-test-go-app-service
            port:
              number: 80
```

## Testing Use Cases

This application is perfect for testing:

### 1. **Kubernetes Installation Verification**
Deploy the app to verify your K8s cluster is working correctly.

```bash
kubectl apply -f deployment.yaml
kubectl get pods -l app=k8s-test-go-app
```

### 2. **Service Discovery Testing**
Test internal service communication and DNS resolution.

```bash
kubectl exec -it <pod-name> -- curl k8s-test-go-app-service
```

### 3. **Load Balancing Verification**
With multiple replicas, verify traffic distribution across pods.

```bash
# Multiple requests should show different pod hostnames
for i in {1..10}; do curl http://test-app.local; done
```

### 4. **Ingress Controller Testing**
Verify ingress controllers are routing traffic correctly.

```bash
curl -H "Host: test-app.local" http://<ingress-ip>
```

### 5. **Rolling Updates**
Test deployment strategies and zero-downtime updates.

```bash
kubectl set image deployment/k8s-test-go-app k8s-test-go-app=k8s-test-go-app:v2
kubectl rollout status deployment/k8s-test-go-app
```

### 6. **Health Checks & Probes**
Add liveness and readiness probes to test pod health monitoring.

### 7. **Network Policies**
Test network segmentation and security policies.

### 8. **Horizontal Pod Autoscaling (HPA)**
Test auto-scaling based on CPU/memory metrics.

## 🔗 Related Resources

For a complete, production-ready Go application deployment guide with advanced Kubernetes features, refer to:
**[DevOps GO APP Repository](https://github.com/abcofdevops/devops-GO-APP)**

This comprehensive repository includes:
- CI/CD pipelines
- Advanced Kubernetes manifests
- Monitoring and logging setup
- Security configurations
- Production best practices

## Customization

### Environment Variables

You can extend the application to use environment variables:

```go
// Add to main.go
port := os.Getenv("PORT")
if port == "" {
    port = "8080"
}
```

### Health Endpoints

Add health check endpoints:

```go
// Add to main.go
http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    fmt.Fprint(w, "OK")
})
```

## 📝 Contributing

This is part of the kubernetes-tools repository. Feel free to submit issues or pull requests for improvements.

## 📄 License

This project is part of the abcofdevops/kubernetes-tools repository. Please refer to the main repository for license information.

---

**Perfect for**: Kubernetes beginners, DevOps engineers testing cluster setups, CI/CD pipeline validation, and anyone learning Kubernetes concepts through hands-on practice.
