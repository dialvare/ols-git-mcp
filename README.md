# Sample App MCP - GitOps Demo

This repository demonstrates a GitOps workflow with OpenShift Lightspeed, MCP servers, and ArgoCD.

## Repository Structure

```
.
├── base/
│   ├── kustomization.yaml
│   └── sample-app-mcp.yaml       # Base application manifest
├── overlays/
│   └── prod/
│       └── kustomization.yaml    # Production-specific configuration
├── gitops-application.yaml       # ArgoCD Application manifest
└── README.md
```

## Deployment

### 1. Deploy ArgoCD Application

```bash
oc apply -f gitops-application.yaml
```

This will configure ArgoCD to automatically sync from this repository's `overlays/prod` path to your OpenShift cluster.

### 2. Automated Sync

ArgoCD is configured with automated sync policies:
- **Auto-sync**: Changes pushed to the `main` branch are automatically deployed
- **Self-heal**: Manual changes in the cluster are reverted to match the Git state
- **Prune**: Resources removed from Git are deleted from the cluster

## Demo Workflow

1. **Initial deployment**: The app is deployed with an intentional error (typo in nodeSelector: `wrker` instead of `worker`)
2. **Incident detection**: OpenShift Lightspeed with Incident Detection MCP identifies the scheduling issue
3. **Fix via GitHub MCP**: OpenShift Lightspeed opens a PR to fix the typo
4. **Merge PR**: Once merged, ArgoCD automatically applies the fix to the cluster
5. **Pod schedules successfully**: The application is now running correctly

## Intentional Error

The `sample-app-mcp.yaml` contains an intentional typo on line 35:
```yaml
nodeSelector:
  node-role.kubernetes.io/wrker: ""  # Should be "worker"
```

This error prevents the pod from being scheduled, triggering an incident that can be detected and fixed through the MCP workflow.
