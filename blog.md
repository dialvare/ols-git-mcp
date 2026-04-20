# Streamlining GitOps workflows with MCPs in OpenShift Lightspeed

Picture this: you're an application developer managing workloads in OpenShift and you want to streamline your daily workflow. You need to monitor your workloads' health, detect incidents as they happen, fix them directly in your source of truth—GitHub—and have those changes automatically applied to your cluster through GitOps. All from a single platform, your OpenShift cluster, without jumping between tools.

That's exactly what I set out to build and what I will show you in this article. By connecting the GitHub MCP Server and the Incident Detection MCP Server to OpenShift Lightspeed, I created a workflow where I can check my workloads' health, identify issues, push fixes to my repo, and let GitOps handle the rest, all powered by AI.

To give you some context, I'm working with an OpenShift 4.21 cluster where I have OpenShift Lightspeed up and running, connected to an LLM (gpt-4). I also have GitOps set up to deploy my workloads straight from my GitHub repo, so whenever I push a change, it gets applied automatically. This GitHub repository contains the following resources:

* **gitops-application.yaml** - GitOps Application custom resource for OpenShift to automatically deploy and track the resources contained in the *manifests* folder
* **/manifests/namespace.yaml** - Namespace (git-apps) where the sample application and related resources lives
* **/manifests/sample-app.yaml** - A sample application, missconfigured with a wrong nodeSelector. Note that it also contains the annotations with the Git repo info
* **/manifests/alerting-rule.yaml** - Custom PrometheusRule that triggers alerts when any app in git-apps namespace is not Running for more than 2 mins

With that foundation in place and my sample app already deployed in the cluster, I'm ready to start exploring how to bring MCPs into the mix.

## Incident Detection MCP Server

If you've ever been on-call, you know the pain of alert storms—dozens of alerts firing at once, and you're left trying to figure out which ones actually matter and what's the root cause. That's exactly the problem that the Incident Detection feature in the Cluster Observability Operator solves. It groups related alerts into incidents, shows them on a timeline color-coded by severity, and categorizes them by affected component. Instead of drowning in noise, you get a clear picture of what's happening. 

Now, the interesting part: there's an MCP server that exposes this functionality. Below you can find the steps I had to follow to install the Cluster Observability Operator, enable the Indidents feature and deploy the MCP server:

1. [Install Red Hat OpenShift Cluster Observability Operator](https://docs.redhat.com/en/documentation/red_hat_openshift_cluster_observability_operator/1-latest/html/installing_red_hat_openshift_cluster_observability_operator/installing-the-cluster-observability-operator#installing-the-cluster-observability-operator-in-the-web-console-_installing_the_cluster_observability_operator)
2. [Install the Monitoring UI plugin](https://docs.redhat.com/en/documentation/red_hat_openshift_cluster_observability_operator/1-latest/html/ui_plugins_for_red_hat_openshift_cluster_observability_operator/monitoring-ui-plugin#coo-monitoring-ui-plugin-install_monitoring-ui-plugin)
3. Deploy the Incident Detection MCP server directly into the OpenShift cluster:

    ```bash
    oc apply -f https://raw.githubusercontent.com/openshift/cluster-health-analyzer/refs/heads/mcp-dev-preview/manifests/mcp/01_service_account.yaml
    oc apply -f https://raw.githubusercontent.com/openshift/cluster-health-analyzer/refs/heads/mcp-dev-preview/manifests/mcp/02_deployment.yaml
    oc apply -f https://raw.githubusercontent.com/openshift/cluster-health-analyzer/refs/heads/mcp-dev-preview/manifests/mcp/03_mcp_service.yaml
    ```

4. A new Incidents dashboard becomes available in our OpenShift cluster under *Observe* > *Alerting* > *Incidents*

For a more detailed explanation on how to configure the full stack, please check [this blog](https://developers.redhat.com/articles/2025/04/15/incident-detection-openshift-tech-preview-here#why_you_need_incident_detection_).

## GitHub MCP Server

Now that the Incident Detection MCP server is deployed, it's time to start with the second component: the [Remote GitHub MCP Server](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md). This MCP server will provide different toolsets to allow us to manage our GitHub workflows remotely, meaning that there's no need to host the server either locally neither in the cluster. However, in order to connect to the remote server we need to authenticate. Here's the configuration needed:

1. In your GitHub repository, navigate to [Personal Access Token settings](https://github.com/settings/personal-access-tokens). There, create your PAT classic token.
2. In our cluster, this PAT needs to be securetly stored as a secret, so it can be used later to connect the MCP to our OpenShift Lightspeed instance:

    ```bash
    oc create secret generic github-mcp-token --from-literal=header="Bearer <your-gh-token>" -n openshift-lightspeed
    ```

## Configure OpenShift Lightspeed

