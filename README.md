# Managing a Spring Application using Docker and Kubernetes

This repository contains the Kubernetes manifests, Minikube configurations, deployment descriptors, service specifications, and orchestration commands for deploying, scaling, and self-healing a containerized Spring Boot application (**NoteVault**) on a Kubernetes cluster.

---

## Architecture Overview

- **Cluster Runtime**: Minikube (v1.39.0 with Docker driver on Windows 11)
- **Container Image**: `sudharshanbunny1/spring-app:v1`
- **Controller**: Kubernetes Deployment (`apps/v1`) managing ReplicaSets and Pod lifecycle
- **Service**: Kubernetes Service (`v1`) with `NodePort` mapping port `8082` to external node port `30080`
- **Application Port**: `8082` (Container) &rarr; `9090` (Local Port-forwarding)

---

## Manifests

### 1. Deployment Specification (`k8s/deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
        - name: spring-app
          image: sudharshanbunny1/spring-app:v1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8082
```

### 2. Service Specification (`k8s/service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  type: NodePort
  selector:
    app: spring-app
  ports:
    - protocol: TCP
      port: 8082
      targetPort: 8082
      nodePort: 30080
```

---

## Step-by-Step Execution Workflow

### 1. Minikube Setup & Cluster Initialization
```powershell
$env:Path += ";C:\minikube"
minikube start --driver=docker
minikube status
```

### 2. Loading Docker Image into Cluster
```powershell
minikube image load sudharshanbunny1/spring-app:v1
minikube image ls
```

### 3. Deploying Application & Exposing Service
```powershell
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods
kubectl get deployments
kubectl get endpointslices -l kubernetes.io/service-name=spring-app-service
```

### 4. Accessing Application via Port-Forwarding
```powershell
kubectl port-forward service/spring-app-service 9090:8082
# Access at: http://localhost:9090
```

### 5. Horizontal Scaling
```powershell
# Scale deployment to 2 replicas
kubectl scale deployment spring-app --replicas=2
kubectl get pods -w
```

### 6. Self-Healing & Resilience Test
```powershell
# Delete a running pod to trigger automatic recovery by Kubernetes ReplicaSet
kubectl delete pod <POD-NAME>
kubectl get pods -l app=spring-app
```

### 7. Stopping the Cluster
```powershell
minikube stop
minikube status
```
