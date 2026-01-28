# OPA Gatekeeper Demo

Demo project for using Open Policy Agent (OPA) Gatekeeper with Kubernetes.

## Prerequisites

- Kubernetes cluster (minikube, kind, or cloud-based)
- kubectl configured to access your cluster
- Helm (optional, for Gatekeeper installation)

## Setup

### 1. Install Gatekeeper

Using Helm:
```bash
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm install gatekeeper/gatekeeper --name-template=gatekeeper --namespace gatekeeper-system --create-namespace
```

Or using kubectl:
```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/v3.14.0/deploy/gatekeeper.yaml
```

### 2. Verify Installation

```bash
kubectl get pods -n gatekeeper-system
```

Wait until all pods are running.

### 3. Apply Constraint Templates

```bash
kubectl apply -f policies/templates/
```

### 4. Apply Constraints

```bash
kubectl apply -f policies/constraints/
```

## Testing the Policies

### Test Valid Resources

These should be accepted:
```bash
kubectl apply -f examples/valid/ --dry-run=server
```

### Test Invalid Resources

These should be rejected:
```bash
# Missing required labels
kubectl apply -f examples/invalid/missing-labels.yaml --dry-run=server

# Privileged container
kubectl apply -f examples/invalid/privileged-pod.yaml --dry-run=server

# No resource limits
kubectl apply -f examples/invalid/no-limits.yaml --dry-run=server

# Unauthorized registry
kubectl apply -f examples/invalid/bad-registry.yaml --dry-run=server
```

## Policies Included

| Policy | Type | Description |
|--------|------|-------------|
| Required Labels | Compliance | Require `team` label on Deployments |
| Allowed Registries | Compliance | Only allow images from approved registries |
| Block Privileged | Security | Deny privileged containers |
| Resource Limits | Compliance | Require CPU/memory limits |

## Cleanup

```bash
kubectl delete -f policies/constraints/
kubectl delete -f policies/templates/
helm uninstall gatekeeper -n gatekeeper-system
kubectl delete namespace gatekeeper-system
```
