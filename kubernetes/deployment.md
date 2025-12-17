# Kubernetes Deployment Guide

This repository ships Kubernetes manifests in `kubernetes/manifests` for all services. Use this guide to build images, push them to a registry, update the manifests, and deploy everything at once.

## Prerequisites
- A working Kubernetes cluster and `kubectl` with the correct context set.
- Docker/Podman to build images.
- Access to a container registry (e.g., GHCR/ACR/ECR/GCR) and permissions to push/pull.
- Optional: a namespace (this guide uses `insurance`).

## 1) Build and push images
Run this per service from its directory, substituting your registry, project, and tag.

```bash
REGISTRY=<registry>/<project>
TAG=<tag>

# Example for customer-core
cd customer-core
docker build -t "$REGISTRY/customer-core:$TAG" .
docker push "$REGISTRY/customer-core:$TAG"

# Repeat for other services
# customer-management-backend
# customer-management-frontend
# customer-self-service-backend
# customer-self-service-frontend
# policy-management-backend
# policy-management-frontend
# risk-management-server
# spring-boot-admin
```

## 2) Update manifests with image tags
Edit each `image:` field in `kubernetes/manifests/*.yaml` to match the pushed image references, e.g.:

```yaml
image: <registry>/<project>/customer-core:<tag>
```

## 3) Create the namespace (once)

```bash
kubectl create namespace insurance || true
```

## 4) Apply all manifests

```bash
kubectl apply -n insurance -f kubernetes/manifests/
```

## 5) Verify rollout

```bash
kubectl get pods,svc,ing -n insurance
kubectl rollout status deployment/<name> -n insurance
```

Check logs or descriptions if anything is Pending or CrashLoopBackOff:

```bash
kubectl logs <pod> -n insurance
kubectl describe pod <pod> -n insurance
```

## 6) Updating after a new build
- Build and push a new tag.
- Update the `image:` fields (or enable `imagePullPolicy: Always`).
- Re-apply: `kubectl apply -n insurance -f kubernetes/manifests/`
- Watch rollout: `kubectl rollout status deployment/<name> -n insurance`

## 7) Clean up

```bash
kubectl delete -n insurance -f kubernetes/manifests/
# Optionally delete the namespace when done
kubectl delete namespace insurance
```
