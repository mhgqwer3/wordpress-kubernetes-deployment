# WordPress + MySQL Kubernetes Deployment

This project contains Kubernetes manifests for deploying a WordPress application with MySQL database using ClusterIP services.

## Architecture Overview

The deployment consists of the following components:

1. **MySQL Database**
   - Deployment: `mysql-app`
   - Service: `mysql-service` (ClusterIP)
   - PVC: `mysql-pvc`
   - Secret: `mysql-secret`

2. **WordPress Application**
   - Deployment: `wordpress-app`
   - Service: `wordpress-service` (ClusterIP)
   - PVC: `wordpress-pvc`

3. **Storage**
   - StorageClass: `mysql-sc`

## Service Connections

### MySQL Service
- **Name**: `mysql-service`
- **Type**: ClusterIP
- **Port**: 3306
- **Selector**: `app: wordpress, tier: mysql`
- **Purpose**: Provides internal database access to WordPress pods

### WordPress Service
- **Name**: `wordpress-service`
- **Type**: ClusterIP
- **Port**: 80
- **Selector**: `app: wordpress, tier: frontend`
- **Purpose**: Provides internal web access to WordPress pods

### Connection Flow
```
WordPress Pod → mysql-service:3306 → MySQL Pod
```

The WordPress deployment connects to MySQL using the environment variable:
```yaml
WORDPRESS_DB_HOST: mysql-service:3306
```

## Deployment Options

### Option 1: Deploy All at Once
```bash
kubectl apply -f deploy-all.yaml
```

### Option 2: Deploy Individual Components
```bash
# 1. Storage and Secrets
kubectl apply -f SC.yaml
kubectl apply -f mysql-secret.yaml
kubectl apply -f PVC.yaml

# 2. Deployments
kubectl apply -f deployment.yaml
kubectl apply -f wordpress-deployment.yaml

# 3. Services
kubectl apply -f services.yaml
```

## Verification

Check if all components are running:
```bash
kubectl get pods
kubectl get services
kubectl get pvc
```

## Accessing WordPress

Since we're using ClusterIP services, WordPress is only accessible within the cluster. To access it externally, you would need to:

1. Create a NodePort or LoadBalancer service
2. Use port-forwarding: `kubectl port-forward service/wordpress-service 8080:80`
3. Access via: `http://localhost:8080`

## Files Structure

- `deploy-all.yaml` - Complete deployment with all components
- `deployment.yaml` - MySQL deployment (fixed syntax errors)
- `wordpress-deployment.yaml` - WordPress deployment
- `services.yaml` - Both ClusterIP services
- `PVC.yaml` - Persistent Volume Claims for both apps
- `mysql-secret.yaml` - Database credentials
- `SC.yaml` - Storage Class configuration
