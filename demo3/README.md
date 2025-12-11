# Demo3: App of Apps Pattern

This demo demonstrates the **App of Apps** pattern in ArgoCD, where a parent Application manages multiple child Applications.

## Structure

```
demo3/
├── app-of-apps/
│   └── parent-app.yml          # Parent Application (manages all child apps)
├── apps/
│   ├── frontend-app.yml        # Child Application for frontend
│   ├── backend-app.yml         # Child Application for backend
│   └── database-app.yml        # Child Application for database
└── workloads/
    ├── frontend/
    │   └── deployment.yml      # Frontend Kubernetes manifests
    ├── backend/
    │   └── deployment.yml      # Backend Kubernetes manifests
    └── database/
        └── deployment.yml      # Database Kubernetes manifests
```

## How It Works

1. **Parent App** (`app-of-apps/parent-app.yml`):
   - Points to `demo3/apps/` directory
   - Creates child Applications in ArgoCD namespace
   - Manages the lifecycle of all child apps

2. **Child Apps** (`apps/*.yml`):
   - Each defines an Application for a specific component
   - Points to respective workload directories
   - Deploys to separate namespaces (frontend, backend, database)

3. **Workloads** (`workloads/*/`):
   - Actual Kubernetes manifests (Deployments, Services, etc.)
   - Deployed and managed by child Applications

## Benefits

- **Centralized Management**: One parent app manages all child apps
- **Modular Structure**: Each component is independently defined
- **Easy Onboarding**: Add new apps by creating a new YAML in `apps/`
- **Namespace Isolation**: Each app deploys to its own namespace
- **GitOps**: All changes tracked in Git

## Usage

### Deploy the App of Apps

```bash
# Apply the parent app to cluster1 (where ArgoCD is running)
kubectl apply -f demo3/app-of-apps/parent-app.yml --context kind-cluster1
```

This will:
1. Create the parent `app-of-apps` Application
2. Parent app syncs and creates 3 child Applications:
   - `frontend-app` → deploys to `frontend` namespace
   - `backend-app` → deploys to `backend` namespace
   - `database-app` → deploys to `database` namespace
3. Each child app syncs and deploys its workloads

### Verify

```bash
# List all Applications
argocd app list

# Check app status
argocd app get app-of-apps
argocd app get frontend-app
argocd app get backend-app
argocd app get database-app

# Check deployed resources
kubectl get pods -n frontend
kubectl get pods -n backend
kubectl get pods -n database
```

### Add a New App

To add a new component:

1. Create workload manifests in `demo3/workloads/new-component/`
2. Create Application YAML in `demo3/apps/new-component-app.yml`
3. Commit and push
4. Parent app auto-syncs and creates the new child app

### Delete Everything

```bash
# Delete parent app (will cascade delete all child apps)
kubectl delete -f demo3/app-of-apps/parent-app.yml --context kind-cluster1
```

## Deploy to Cluster2

To deploy to cluster2 instead, modify the child apps' destination:

```yaml
destination:
  server: https://192.168.147.5:6443  # cluster2
  namespace: frontend
```
