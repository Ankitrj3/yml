# Kubernetes Probes

Kubernetes Probes are mechanisms to monitor the health and readiness of your containers. There are three types of probes that help Kubernetes manage your applications effectively.

## 📋 Types of Probes

### 1. **Liveness Probe**
- **Purpose**: Determines if a container is running properly
- **Action**: If fails, Kubernetes **restarts** the container
- **Use Case**: Detect deadlocks, infinite loops, or crashed applications

### 2. **Readiness Probe**
- **Purpose**: Determines if a container is ready to serve traffic
- **Action**: If fails, removes pod from service endpoints (no traffic sent)
- **Use Case**: Wait for dependencies, initialization, or loading data

### 3. **Startup Probe**
- **Purpose**: Determines if a container has started successfully
- **Action**: If fails, Kubernetes **restarts** the container
- **Use Case**: For slow-starting containers, disables other probes until startup succeeds

## 🔧 Probe Methods

### HTTP GET
```yaml
httpGet:
  path: /health
  port: 8080
  httpHeaders:
  - name: Custom-Header
    value: Awesome
```

### TCP Socket
```yaml
tcpSocket:
  port: 8080
```

### Exec Command
```yaml
exec:
  command:
  - cat
  - /tmp/healthy
```

## ⚙️ Probe Configuration Parameters

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30    # Wait before first probe
  periodSeconds: 10          # How often to perform probe
  timeoutSeconds: 5          # Timeout for probe response
  successThreshold: 1        # Min consecutive successes
  failureThreshold: 3        # Min consecutive failures to trigger action
```

## 📝 Implementation Examples

### Example 1: Web Application with All Probes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  namespace: probes
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:latest
        ports:
        - containerPort: 80
        
        # Startup Probe - Runs first
        startupProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 10
          successThreshold: 1
        
        # Liveness Probe - Monitors if container is alive
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        
        # Readiness Probe - Monitors if container is ready for traffic
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
          successThreshold: 1
```

### Example 2: Database with TCP Probe

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database-deployment
  namespace: probes
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        ports:
        - containerPort: 3306
        
        # Startup Probe - Database initialization can be slow
        startupProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 15
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 10
        
        # Liveness Probe - Check if MySQL is responding
        livenessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Readiness Probe - Check if MySQL is ready for connections
        readinessProbe:
          exec:
            command:
            - bash
            - -c
            - "mysqladmin ping -h localhost -u root -p$MYSQL_ROOT_PASSWORD"
          initialDelaySeconds: 20
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

### Example 3: Custom Application with Exec Probe

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-app-deployment
  namespace: probes
spec:
  replicas: 1
  selector:
    matchLabels:
      app: custom-app
  template:
    metadata:
      labels:
        app: custom-app
    spec:
      containers:
      - name: app
        image: busybox:latest
        command:
        - sh
        - -c
        - |
          # Create health check file after initialization
          sleep 30
          touch /tmp/healthy
          touch /tmp/ready
          # Keep container running
          while true; do sleep 3600; done
        
        # Startup Probe - Check if app has started
        startupProbe:
          exec:
            command:
            - test
            - -f
            - /tmp/healthy
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 10
        
        # Liveness Probe - Check if app is still alive
        livenessProbe:
          exec:
            command:
            - test
            - -f
            - /tmp/healthy
          initialDelaySeconds: 35
          periodSeconds: 10
          timeoutSeconds: 2
          failureThreshold: 3
        
        # Readiness Probe - Check if app is ready
        readinessProbe:
          exec:
            command:
            - test
            - -f
            - /tmp/ready
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 3
```

## 🚀 How to Deploy and Test

### 1. Create Namespace
```yaml
# namespace.yml
apiVersion: v1
kind: Namespace
metadata:
  name: probes
```

### 2. Deploy Examples
```bash
# Apply namespace
kubectl apply -f namespace.yml

# Deploy any of the examples above
kubectl apply -f webapp-deployment.yml
kubectl apply -f database-deployment.yml
kubectl apply -f custom-app-deployment.yml
```

### 3. Monitor Probes
```bash
# Check pod status
kubectl get pods -n probes

# Describe pod to see probe events
kubectl describe pod <pod-name> -n probes

# Watch pod events in real-time
kubectl get events -n probes --watch

# Check pod logs
kubectl logs <pod-name> -n probes
```

## 📊 Testing Probe Scenarios

### Test Liveness Probe Failure
```bash
# Exec into pod and simulate failure
kubectl exec -it <pod-name> -n probes -- rm /tmp/healthy

# Watch pod get restarted
kubectl get pods -n probes --watch
```

### Test Readiness Probe Failure
```bash
# Exec into pod and simulate not ready
kubectl exec -it <pod-name> -n probes -- rm /tmp/ready

# Check endpoints (pod should be removed)
kubectl get endpoints -n probes
```

### Monitor Probe Status
```bash
# Get detailed probe information
kubectl get pods -n probes -o wide

# Check probe results in pod description
kubectl describe pod <pod-name> -n probes | grep -A 10 "Conditions"
```

## 🔍 Troubleshooting Probes

### Common Issues

1. **Probe Timeout**
   ```bash
   # Increase timeoutSeconds
   timeoutSeconds: 10
   ```

2. **Probe Too Aggressive**
   ```bash
   # Increase periodSeconds and failureThreshold
   periodSeconds: 15
   failureThreshold: 5
   ```

3. **Slow Startup**
   ```bash
   # Use startup probe with higher failure threshold
   startupProbe:
     failureThreshold: 20
     periodSeconds: 10
   ```

### Debug Commands
```bash
# Check probe configuration
kubectl get pod <pod-name> -n probes -o yaml | grep -A 20 "livenessProbe\|readinessProbe\|startupProbe"

# Monitor probe events
kubectl get events -n probes --field-selector involvedObject.name=<pod-name>

# Check pod restart count
kubectl get pods -n probes -o wide
```

## 📈 Best Practices

### 1. **Startup Probe**
- Use for slow-starting containers
- Set high `failureThreshold` and reasonable `periodSeconds`
- Disable other probes until startup succeeds

### 2. **Liveness Probe**
- Check application core functionality
- Avoid expensive operations
- Use different endpoint than readiness if possible

### 3. **Readiness Probe**
- Check dependencies (database, external services)
- Use for graceful shutdowns
- More frequent than liveness probe

### 4. **General Guidelines**
- Start with reasonable delays (`initialDelaySeconds`)
- Don't make probes too sensitive (`failureThreshold >= 3`)
- Use appropriate timeouts (`timeoutSeconds`)
- Monitor probe metrics and adjust accordingly

## 🧹 Cleanup

```bash
# Delete all resources
kubectl delete namespace probes
```

## 📚 Probe Decision Matrix

| Scenario | Startup | Liveness | Readiness |
|----------|---------|----------|-----------|
| App initialization | ✅ | ❌ | ❌ |
| App crashed/deadlocked | ❌ | ✅ | ❌ |
| Temporary overload | ❌ | ❌ | ✅ |
| Dependency unavailable | ❌ | ❌ | ✅ |
| Graceful shutdown | ❌ | ❌ | ✅ |
| Memory leak detection | ❌ | ✅ | ❌ |