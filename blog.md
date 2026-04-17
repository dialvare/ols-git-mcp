# Streamlining GitOps workflows with MCPs in OpenShift Lightspeed

Picture this: you're an application developer managing workloads in OpenShift and you want to streamline your daily workflow. You need to monitor your workloads' health, detect incidents as they happen, fix them directly in your source of truth—GitHub—and have those changes automatically applied to your cluster through GitOps. All from a single platform, your OpenShift cluster, without jumping between tools.

That's exactly what I set out to build and what I will show you in this article. By connecting the GitHub MCP Server and the Incident Detection MCP Server to OpenShift Lightspeed, I created a workflow where I can check my workloads' health, identify issues, push fixes to my repo, and let GitOps handle the rest, seamlessly, all powered by AI models.

To give you some context, I was already working with an OpenShift 4.20 cluster where I had OpenShift Lightspeed up and running, connected to an LLM (gpt-4). I also had GitOps set up to deploy my workloads straight from my GitHub repo—so whenever I push a change, it gets applied automatically. With that foundation in place, I was ready to start exploring how to bring MCPs into the mix.

## Incident Detection MCP Server

If you've ever been on-call, you know the pain of alert storms—dozens of alerts firing at once, and you're left trying to figure out which ones actually matter and what's the root cause. That's exactly the problem that the Incident Detection feature in the Cluster Observability Operator solves. It groups related alerts into incidents, shows them on a timeline color-coded by severity, and categorizes them by affected component. Instead of drowning in noise, you get a clear picture of what's happening.

Now, the interesting part: there's an MCP server that exposes this functionality. This means I can ask Lightspeed things like "Are there any active incidents in my cluster?" or "What's causing the critical alerts right now?"—and get answers without leaving the conversation. It felt like having a second pair of eyes watching over my workloads.

Steps:

1. [Install Red Hat OpenShift Cluster Observability Operator](https://docs.redhat.com/en/documentation/red_hat_openshift_cluster_observability_operator/1-latest/html/installing_red_hat_openshift_cluster_observability_operator/installing-the-cluster-observability-operator#installing-the-cluster-observability-operator-in-the-web-console-_installing_the_cluster_observability_operator)
2. [Install the Monitoring UI plugin](https://docs.redhat.com/en/documentation/red_hat_openshift_cluster_observability_operator/1-latest/html/ui_plugins_for_red_hat_openshift_cluster_observability_operator/monitoring-ui-plugin#coo-monitoring-ui-plugin-install_monitoring-ui-plugin)
3. Deploy the Incident Detection MCP server:

```bash
oc apply -f https://raw.githubusercontent.com/openshift/cluster-health-analyzer/refs/heads/mcp-dev-preview/manifests/mcp/01_service_account.yaml
oc apply -f https://raw.githubusercontent.com/openshift/cluster-health-analyzer/refs/heads/mcp-dev-preview/manifests/mcp/02_deployment.yaml
oc apply -f https://raw.githubusercontent.com/openshift/cluster-health-analyzer/refs/heads/mcp-dev-preview/manifests/mcp/03_mcp_service.yaml
```

For a more detailed explanation on how to configure the full Monitoring stack, please check [this blog](https://developers.redhat.com/articles/2025/04/15/incident-detection-openshift-tech-preview-here#why_you_need_incident_detection_)

## GitHub MCP Server
