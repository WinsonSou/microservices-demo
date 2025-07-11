# High Availability Deployment Instructions

## Overview
This deployment has been configured for high availability with:
- Multiple replicas for all services
- Redis Sentinel for automatic failover
- Pod Disruption Budgets for safe updates
- Anti-affinity rules to spread pods across nodes
- Persistent volumes for Redis data

## Prerequisites
- Kubernetes cluster with at least 3 worker nodes
- `nutanix-volume` StorageClass available
- Sufficient resources for ~30 pods

## Deployment Order

Deploy the resources in this specific order:

```bash
# Set kubeconfig
export KUBECONFIG=/Users/winson.sou/wskn-nkp-mgmt.conf

# 0. Create namespace first
kubectl create namespace microservices-demo

# 1. Deploy Redis StatefulSet first (includes PVCs)
kubectl apply -f redis-statefulset.yaml

# 2. Wait for Redis pods to be ready
kubectl wait --for=condition=ready pod -l app=redis -n microservices-demo --timeout=300s

# 3. Deploy Redis Sentinel
kubectl apply -f redis-sentinel.yaml

# 4. Wait for Sentinel pods to be ready
kubectl wait --for=condition=ready pod -l app=redis-sentinel -n microservices-demo --timeout=300s

# 5. Deploy all microservices
kubectl apply -f kubernetes-manifests.yaml

# 6. Deploy Pod Disruption Budgets
kubectl apply -f pod-disruption-budgets.yaml
```

## Verification

Check deployment status:
```bash
# Check all pods are running
kubectl get pods -n microservices-demo

# Check Redis Sentinel status
kubectl exec -it redis-sentinel-0 -n microservices-demo -- redis-cli -p 26379 sentinel masters

# Check PVCs
kubectl get pvc -n microservices-demo

# Check services
kubectl get svc -n microservices-demo

# Check PDBs
kubectl get pdb -n microservices-demo
```

## Access the Application

```bash
# Get the LoadBalancer IP
kubectl get svc frontend-external -n microservices-demo

# Or use port-forward for local access
kubectl port-forward svc/frontend -n microservices-demo 8080:80
```

## Configuration Summary

### Service Replicas:
- **Critical Services (3 replicas)**: frontend, cartservice, checkoutservice, productcatalogservice
- **Standard Services (2 replicas)**: All other services
- **Redis**: 3 instances (1 master, 2 replicas) with 10GB PVCs each
- **Redis Sentinel**: 3 instances for monitoring and failover

### Pod Disruption Budgets:
- Critical services: minAvailable=2
- Standard services: minAvailable=1
- Redis & Sentinel: minAvailable=2

### High Availability Features:
- Pod anti-affinity to spread across nodes
- Redis Sentinel for automatic failover
- Persistent volumes for Redis data
- CartService configured to use Sentinel connection

## Troubleshooting

If Redis Sentinel connection fails:
```bash
# Check Sentinel logs
kubectl logs -l app=redis-sentinel -n microservices-demo

# Test Sentinel connectivity from cartservice
kubectl exec -it deployment/cartservice -n microservices-demo -- nc -zv redis-sentinel-0.redis-sentinel 26379
```

## Cleanup

To remove all resources:
```bash
kubectl delete -f pod-disruption-budgets.yaml
kubectl delete -f kubernetes-manifests.yaml
kubectl delete -f redis-sentinel.yaml
kubectl delete -f redis-statefulset.yaml
kubectl delete pvc -l app=redis -n microservices-demo

# Or delete the entire namespace
kubectl delete namespace microservices-demo
```