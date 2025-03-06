+++
title = "Using Workshop Studio"
weight = 12
+++
If you're  running these labs as part of an AWS led event in Workshop Studio, you don't need to provision anything since all the resources needed to complete all modules will be pre provisioned.

## What will be provisioned in your account?
The stack sets up the following components:

1. **EKS Cluster**:
    -   Provisions a new EKS cluster named opea-eks-cluster.
    -   Deploys a node within the cluster using an M7i.24xlarge instance.
2. **CloudFormation Templates**:
Creates templates for the following modules:
    -   Module 1: ChatQnA **Default**
    -   Module 2: ChatQnA with Guardrails
    -   Module 3: ChatQnA with OpenSearch (open source) as the vector database
    -   Module 4: ChatQnA with Bedrock as the LLM
    -   Module 5: ChatQnA with Remote Inference (Denvr) as the LLM

# Step 1: Configure your access to the cluster

You will interact with your EKS cluster using `kubectl`, you need to configure your config file to recognize the cluster. This is done by updating the kubeconfig file, which stores details about cluster authentication and access.

1.  Log in to your AWS Management Console:

![Console](/images/console.png)
 
2. In your console click on the Cloud Shell icon. You can also use your own AWS CLI, in this case you need to be sure to install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) Client and [Kubectl](https://kubernetes.io/docs/tasks/tools/) in your personal environment. 

![Cloud Shell](/images/cloudshell.png)

3. Update Your kubeconfig
Run the following command to update your kubeconfig with your cluster information:

```bash
aws eks update-kubeconfig --name ${clusterName} --region us-east-1
```

You should receive an output confirming your conf file was updated:

```
Updated context arn:aws:eks:us-east-2:<<Account_ID>>:cluster/opea-eks-cluster in /home/cloudshell-user/.kube/config
```

You are now ready to interact with the Kubernetes cluster using `kubectl`

# Step 3: Verify you can access your cluster:
After updating your kubeconfig, check if you can successfully connect to the cluster. A simple test is to see if you can see the nodes associated with the cluster:

```bash
kubectl get nodes
```

If the command is successful, you should see an output similar to this:

```
NAME                            STATUS   ROLES    AGE    VERSION
ip-XXX-XXX-XXX-XXX.ec2.internal   Ready    <none>   146m   v1.27.16-eks-XXXXX
``` 

You are now ready to explore the Module of your preference.