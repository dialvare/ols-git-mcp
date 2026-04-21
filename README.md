# Sample App MCP - GitOps Demo

This repository demonstrates a GitOps workflow with OpenShift Lightspeed, MCP servers, and ArgoCD.

## Repository Structure

```
.
├── manifests/
│   ├── namespace.yaml            # git-apps namespace definition
│   ├── sample-app-mcp.yaml       # Sample Pod with intentional nodeSelector error
│   └── alerting-rule.yaml        # PrometheusRule for incident detection
├── gitops-application.yaml       # ArgoCD Application manifest
├── argocd-rbac.yaml              # RBAC configuration for ArgoCD
└── README.md
```

## Deployment

### 1. Deploy ArgoCD Application

```bash
oc apply -f gitops-application.yaml
```

This will configure ArgoCD to automatically sync from this repository's `manifests/` directory to the `git-apps` namespace in your OpenShift cluster.

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

## Resources Deployed

### Namespace
- **git-apps**: Dedicated namespace for the sample application and monitoring resources

### Sample Application
- **sample-app-mcp**: A simple HTTPD pod with intentional misconfiguration

### Alerting Rules
The `alerting-rule.yaml` defines three PrometheusRules for incident detection:
- **PodNotScheduled**: Fires when a pod is in Pending state for more than 2 minutes
- **PodCrashLooping**: Fires when a pod restarts more than 3 times in 10 minutes
- **PodNotReady**: Fires when a pod is not ready for more than 3 minutes

## Intentional Error

The `manifests/sample-app-mcp.yaml` contains an intentional typo on line 39:
```yaml
nodeSelector:
  node-role.kubernetes.io/wrker: ""  # Should be "worker"
```

This error prevents the pod from being scheduled, triggering the **PodNotScheduled** alert after 2 minutes, which can be detected and fixed through the MCP workflow with OpenShift Lightspeed.
