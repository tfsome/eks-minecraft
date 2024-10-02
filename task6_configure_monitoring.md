# Monitoring (Estimated time: 2h)

## Agenda

In this lab you will add monitoring and logging for our services and collect them on a dashboard.

## Configure CloudWatch ContainerInsights

In this task you will need to enable ContainerInsights for your cluster. To do this you should to install cloud-watch agent in your cluster and configure it.

Several considerations regarding the task:

- CloudWatch Agent requires additional permissions to write logs and send metrics. Consider adding `CloudWatchAgentServerPolicy` build-in policy or similiar custom policy to a `minecraft-worker` IAM role.

- CloudWatch Agent are distributed through several sources like [Docker Hub](https://hub.docker.com/r/amazon/cloudwatch-agent), [Public AWS ECR](https://gallery.ecr.aws/cloudwatch-agent/cloudwatch-agent) and [S3 bucket](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html). As our worker nodes are running with no Internet access we need delivering image to our cluster using some workaround. **Hint**: the easiest way is to retag image to a our ECR repository(create a new repository for agent) and use this private image in a DaemonSet.

- This [repository](https://github.com/aws-samples/amazon-cloudwatch-container-insights/blob/master/k8s-deployment-manifest-templates/deployment-mode/daemonset/) help you to write DaemonSet manifest for CloudWatch Agent. Please get more details regarding deployment in [this document](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html). Please keep in mind that you need to create your deployment as cluster has no internet access.

- Hint: replace variable in configMap with the exact cluster name.

After CloudWatch Agent container up and running wait about 5-10 minutes to get metrics and logs delivered to AWS CloudWatch. After you see new metrics availible in ContainerInsights chapter - you've done with this subtask.

## Configure logs
- Configure CloudWatch agent to send logs for `minecraft-worker` and `minecraft-master` pods to AWS CloudWatch Logs.

## Create metrics dashboard
Create a CloudWatch Dashboard with the several widgets: 
- Logs from minecraft-master (limit it to 16 lines)
- Logs from minecraft-worker (show only the last 5 errors)
- Following metrics for `minecraft-worker` and `minecraft-master` pods:
  - Pod CPU Utilization 
  - Running pods count
  - Pod Memory Utilization
  - Pod Rx and Tx bytes
- Number of running pods in a minecraft namespace
- Following metrics for EKS cluster:
  - Number of failed nodes in cluster
  - Number the whole nodes in cluster
  - Count of EC2 instances in ASG (managed by EKS node group)
  - Average EC2 CPU Utilization for ASG (managed by EKS node group)
- Health host counter for NLB. Then you should configure alert with this metric which will triggers when there are less then 2 nodes.

## Test dashboard
Open your dashboard and have a test play in Minecraft. Observe metrics like CPU and Network and differ values under the load and during the servers idle.

## (Optional) Configure HPA and Cluster Autoscaler
As and optional task you can confugure autoscaling for your EKS Managed group using the [Kubernetes HPA and cluster Autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html). Choose the CPU utilization value what is lower then usual during the playing and configure it as a triger for scale-out. Then play in Minecraft for 5 minutes, and find your dashboard if there new nodes appeared. Another option is to generate load with some [other way](https://serverfault.com/questions/1076153/load-testing-cpu-utilization-that-occurred-while-executing-at-the-system-level).

## Definition of done

You should provide the output of the following script as the proof of working solution:

```bash
DASHBOARD={your_dashboard_name}
REGION={aws_region} 
aws cloudwatch get-dashboard --dashboard-name $DASHBOARD --region $REGION | jq -c '.DashboardBody | fromjson' | jq '.widgets[] | [.properties.title, .type]'
```


## Clean-up

Do not forget to stop and delete your resources on the end of practice. Please keep in mind that log group and metrics are keep even after the cluster removal. You can delete them mannualy.
You can use Tags to locate required resources.

