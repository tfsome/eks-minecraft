# Create EKS cluster

## Agenda

In this lab we will create EKS cluster.

To achieve this we will need to create some AWS resources: 
- Launch template for EKS managed node group's "worker nodes"
- Elastic Kubernetes Service for running our game server
- EKS managed node group

## Create VPC endpoints

**Cost optimization note:** Please keep in mind that VPC Endpoints _has no free tier and will charge you_.<br>

EKS worker nodes will be running in Private subnets and cannot access AWS services via Internet.

Therefore you have to configure VPC Endpoints for the following services: `EC2`, `ECR`, `STS`, `S3`, `CloudWatch` and `CloudWatch logs` services. See details [here](https://docs.aws.amazon.com/eks/latest/userguide/private-clusters.html)

You have to assign all interface type VPC endpoints with `{vpc_endpoints}` security group.

Gateway type VPC endpoint should be assigned with private network routing table.


## Create Launch Template
We will use launch template for more customization of our node group.
Create launch template for your worker nodes with the following parameters:
- `name`=`minecraft-worker-node`
- `delete_on_termination`=`true` (for networking and for storage resources)
- `Security groups`={eks_worker security group}  under `Network settings`
- `instance_type` = `t3.medium`
- `Key pair name` (optional)

## Create EKS cluster
**Cost optimization note:** Please keep in mind that EKS _has no free tier and will charge you_.<br>

And you have to create EKS cluster:
- `name`=`minecraft-eks`
- `role`=`{eks_master}`
- `private access endpoint`=`true`
- `subnets`=`{private_subnets_id, public_subnets_id}`
- `security groups`=`{eks_master}`

## Create EKS managed node group
Create EKS managed node group using two Private subnets:
- `name`=`minecraft-eks-node-group`
- `cluster`=`minecraft-eks`
- `role`=`{eks_worker}`
- `Minimum size`=`1`
- `Maximum size`=`2`
- `subnet_ids`=`{private_subnets_id}`
- `ami_type`=`AL2_x86_64`
- `launch template`=`minecraft-worker-node`

Refer to [documentation](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html) in case of any questions.

## Definition of done

Your EKS cluster and managed node group created successfully. The nodes have connected to the EKS cluster.

## Clean-up

Do not forget to stop and delete your resources on the end of practice. You can use Tags to locate required resources.

