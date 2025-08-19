# Kubernetes Ingress Project

This project demonstrates Kubernetes Ingress with NGINX Ingress Controller using two sample applications: NGINX and BusyBox.

## 📁 Project Structure

```
ingressK8s/
├── appDeployment.yml     # BusyBox deployment (serves on port 5000)
├── appService.yml        # BusyBox service
├── nginxDeployment.yml   # NGINX deployment (serves on port 80)
├── nginxService.yml      # NGINX service (exposed on port 3000)
├── ingress.yml          # Ingress configuration
└── namespace.yml        # Namespace definition
```

## 🚀 Prerequisites

- **Minikube** or any Kubernetes cluster
- **kubectl** configured
- **NGINX Ingress Controller** installed

## 📋 Setup Instructions

### 1. Start Minikube
```bash
minikube start
```

### 2. Enable NGINX Ingress Controller
```bash
minikube addons enable ingress
```

### 3. Deploy the Application
```bash
# Deploy all resources
kubectl apply -f .

# Verify deployments
kubectl get pods -n ingress
kubectl get svc -n ingress
kubectl get ingress -n ingress
```

## 🌐 How to Access in Browser

### Method 1: Using Minikube Tunnel (Recommended)
```bash
# Start the tunnel (requires sudo password)
minikube tunnel
```
- **Root Path**: http://localhost/ → BusyBox application
- **NGINX Path**: http://localhost/nginx → NGINX application

### Method 2: Using Minikube IP + NodePort
```bash
# Get Minikube IP and NodePort
minikube ip
kubectl get svc ingress-nginx-controller -n ingress-nginx -o jsonpath='{.spec.ports[0].nodePort}'

# Access using: http://<MINIKUBE_IP>:<NODEPORT>/
# Example: http://192.168.49.2:31793/
```

### Method 3: Using Minikube Service Command
```bash
minikube service ingress-nginx-controller -n ingress-nginx
```

## 🔧 How to Test Using Terminal (curl)

### Using Minikube Tunnel
```bash
# Start tunnel first
minikube tunnel

# Test root path (BusyBox)
curl http://localhost/

# Test nginx path
curl http://localhost/nginx

# Verbose output for debugging
curl -v http://localhost/
curl -v http://localhost/nginx
```

### Using Minikube IP + NodePort
```bash
# Get variables
MINIKUBE_IP=$(minikube ip)
NODEPORT=$(kubectl get svc ingress-nginx-controller -n ingress-nginx -o jsonpath='{.spec.ports[0].nodePort}')

# Test endpoints
curl http://$MINIKUBE_IP:$NODEPORT/
curl http://$MINIKUBE_IP:$NODEPORT/nginx
```

### Using Port-Forward (Alternative)
```bash
# Port-forward the ingress controller
kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80

# In another terminal, test:
curl http://localhost:8080/
curl http://localhost:8080/nginx
```

## 📊 Expected Responses

### Root Path (/)
```bash
curl http://localhost/
# Expected: Hello from BusyBox!
```

### NGINX Path (/nginx)
```bash
curl http://localhost/nginx
# Expected: NGINX welcome page HTML
```

## 🔍 Troubleshooting Commands

### Check Pod Status
```bash
kubectl get pods -n ingress
kubectl describe pod <pod-name> -n ingress
```

### Check Service Endpoints
```bash
kubectl get endpoints -n ingress
kubectl describe svc <service-name> -n ingress
```

### Check Ingress Status
```bash
kubectl get ingress -n ingress
kubectl describe ingress ingress-route -n ingress
```

### Check Ingress Controller Logs
```bash
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

### Test Individual Services (Bypass Ingress)
```bash
# Test BusyBox service directly
kubectl port-forward svc/busybox-service -n ingress 5000:5000
curl http://localhost:5000/

# Test NGINX service directly
kubectl port-forward svc/nginx-service -n ingress 3000:3000
curl http://localhost:3000/
```

## 🛠️ Configuration Details

### Ingress Routes
- **/** → `busybox-service:5000` (BusyBox application)
- **/nginx** → `nginx-service:3000` (NGINX application mapped to port 80)

### Services
- **busybox-service**: ClusterIP, port 5000 → targetPort 5000
- **nginx-service**: ClusterIP, port 3000 → targetPort 80

## 🧹 Cleanup

```bash
# Delete all resources
kubectl delete -f .

# Or delete namespace (removes everything)
kubectl delete namespace ingress
```

## ❓ Common Issues

1. **"Connection refused"**: Check if pods are running and services have endpoints
2. **"404 Not Found"**: Verify ingress paths and service names
3. **"Bad Gateway"**: Check if application pods are healthy and responding
4. **Can't access via localhost**: Use `minikube tunnel` or direct IP access
