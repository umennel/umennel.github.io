---
layout: post
author: Uche Mennel
---

Once you have your critical application running on a Kubernetes Cluster, it is important to monitor its performance metrics in order to sustain stable operation.
Without enough visibility, you will never be able to identify resource bottlenecks or determine the maximum load that the cluster can sustain. In case of failure, it is might be necessary to drill down into log messages to find the root cause.

I looked at two different approaches, specific to monitoring Azure Kubernetes Services workloads:
  * [Azure Monitor for Containers][1]
  * [Kubernetes Dashboard][7]


## Enable Azure Monitor for Containers

[Azure Monitor for Containers][1] is based on [Log Analytics Workspace][5], which integrates with the [Prometheus][9] Metrics Server deployed on the Kubernetes Cluster. Application and workload metrics are collected from Kubernetes nodes using queries and can be used to create custom alerts, dashboards, and detailed performance analysis.
There are several ways to set up monitoring of your Kubernetes workload in Azure. The easiest is go to your Cluster and enable Insights

![Enable Insights](/assets/enable-insights.png)

This starts an onboarding process which will guide you through the necessary steps of creating a [Log Analytics Workspace][5] (on choosing an existing one) and then setting up Log Analytics to make additional metrics available to Azure Monitoring.
Before any data can be retrieved by Azure Monitoring, a Log Analytics Agent has to be deployed on the targeted Cluster. The agent provides performance metrics via a [Prometheus][9] Metrics Service. The instructions for deploying the agent can be found [here][4] for Linux and [here][5] for Windows respectively. If you therefore choose to deploy a Kubernetes DaemonSet without secrets, you will have to provide the Workspace ID and Key of your Log Analytics Workspace. You will find these credentials under *Log Analytics Workspace → Agents Management*

![Log Analytics Agent Credentials](/assets/la-credentials.png)

Once your Log Analytics Agent is up and running, performance metrics are available to your Cluster Monitoring service.

Be aware that the Azure Monitoring might come with high cost, depending on the amount of data that is transferred. I recommend to check [pricing][6] first, before taking on this approach.

## Use the Kubernetes Dashboard

If you just need a simple overview dashboard for your Kubernetes Cluster Performance, the Kubernetes Dashboard might come in handy. In contrast to Azure Monitoring, it is very convenient to use, and on top of that, it is free.
If using AKS, the metrics server and dashboard are already installed on every Kubernetes Cluster.
Otherwise, the installation process is described [here][7]. It basically boils down to executing the following command,

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

After successful installation, you will find the *kubernetes-dashboard* URL with the *cluster-info* command, i.e.

    kubectl cluster-info

The dashboard can be made available on your local machine via [kubectl proxy][8].

The Kubernetes proxy will establish an authenticated connection to the Cluster IP running on the Kubernetes Cluster, so that you can access the dashboard with your web browser. The URL is obtained by replacing the Cluster host name with the address of your local proxy (localhost:8001 by default).

With AKS, the dashboard is conveniently accessible by using the Azure-Cli *browse* command:

    az aks browse --resource-group myResourceGroup --name myCluster

And your dashboard loads right away inside your default browser.


[1]: https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-overview  "Azure Monitor for Containers Overview"
[2]: https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-onboard "Enable Azure Monitor for containers"
[3]: https://docs.microsoft.com/en-us/azure/azure-monitor/insights/containers#configure-a-log-analytics-linux-agent-for-kubernetes "Configure a Log Analytics Linux agent for Kubernetes"
[4]: https://docs.microsoft.com/en-us/azure/azure-monitor/insights/containers#configure-a-log-analytics-windows-agent-for-kubernetes "Configure a Log Analytics Windows agent for Kubernetes"
[5]: https://docs.microsoft.com/en-gb/azure/azure-monitor/log-query/log-query-overview "Log Analytics Overview"
[6]: https://azure.microsoft.com/en-gb/pricing/details/monitor/ "Azure Monitor Pricing"
[7]: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/ "Kubernetes Dashboard"
[8]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#proxy "Kubernetes Proxy"
[9]: https://prometheus.io/docs/introduction/overview/ "Prometheus Overview"
