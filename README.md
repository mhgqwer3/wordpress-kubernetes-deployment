# WordPress + MySQL Kubernetes Deployment

This project contains Kubernetes manifests for deploying a WordPress application with MySQL database using LoadBalancer services for external access.

## Architecture Overview

The deployment consists of the following components:

1. **MySQL Database**
   - Deployment: `mysql-app`
   - Service: `mysql-svc` (ClusterIP - internal only)
   - PVC: `mysql-pvc` (5Gi EBS storage)
   - Secret: `mysql-secret` (root password)

2. **WordPress Application**
   - Deployment: `wordpress-app`
   - Service: `wordpress-svc` (LoadBalancer - external access)
   - PVC: `wordpress-efs-pvc` (5Gi EFS storage)
   - Image: `wordpress:6.4-apache`

3. **Storage Classes**
   - `mysql-sc`: EBS CSI driver for MySQL (ReadWriteOnce)
   - `efs-sc`: EFS CSI driver for WordPress (ReadWriteMany)

## Service Connections

### MySQL Service
- **Name**: `mysql-svc`
- **Type**: ClusterIP (internal cluster access only)
- **Port**: 3306
- **Selector**: `app: wordpress, tier: mysql`
- **Purpose**: Provides internal database access to WordPress pods

### WordPress Service
- **Name**: `wordpress-svc`
- **Type**: LoadBalancer (external access)
- **Port**: 80
- **Selector**: `app: wordpress, tier: frontend`
- **Purpose**: Provides external web access to WordPress application

### Connection Flow
```
External Users → LoadBalancer → WordPress Pod → mysql-svc:3306 → MySQL Pod
```

The WordPress deployment connects to MySQL using the environment variable:
```yaml
WORDPRESS_DB_HOST: mysql-svc:3306
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

The WordPress application is accessible externally through the LoadBalancer service:

1. **External Access**: The LoadBalancer will provide an external IP address
2. **Check LoadBalancer IP**: `kubectl get service wordpress-svc`
3. **Access WordPress**: `http://EXTERNAL-IP` (replace EXTERNAL-IP with the actual IP)

### Alternative Access Methods:
- **Port Forwarding**: `kubectl port-forward service/wordpress-svc 8080:80`
- **Access via**: `http://localhost:8080`

## Files Structure

- `deploy-all.yaml` - Complete deployment with all components (RECOMMENDED)
- `deployment.yaml` - MySQL deployment
- `wordpress-deployment.yaml` - WordPress deployment
- `services.yaml` - MySQL ClusterIP service
- `wordpress-services.yaml` - WordPress LoadBalancer service
- `PVC.yaml` - MySQL Persistent Volume Claim
- `wordpress-pvc.yaml` - WordPress Persistent Volume Claim
- `wordpress-pv.yaml` - WordPress Persistent Volume
- `mysql-secret.yaml` - Database credentials
- `SC.yaml` - MySQL Storage Class (EBS)
- `wordpress/sc.yaml` - WordPress Storage Class (EFS)

## Prerequisites

- Kubernetes cluster (EKS, GKE, AKS, or local)
- AWS EBS and EFS CSI drivers installed
- kubectl configured to access your cluster
