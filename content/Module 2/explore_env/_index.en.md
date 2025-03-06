+++
title = "Deploy Guardrails"
weight = 200
+++

# Why would you need guardrails?

As AI becomes more integral in applications, ensuring reliable, safe, and ethical outcomes is critical. Guardrails in AI systems, especially those interacting with users or making decisions, help maintain control over responses, avoid unintended behaviors, and align output with set goals and standards. Without guardrails, AI models might generate unsafe or biased content, make inappropriate decisions, or mishandle sensitive data.

Implementing guardrails allows you to:

1. Enhance User Safety – By controlling responses, you reduce the risks of harmful outputs in AI interactions.
2. Maintain Compliance – As privacy and safety regulations grow, guardrails ensure your applications remain within legal boundaries.
3. Optimize Accuracy – Guardrails can improve the quality of AI output, reducing error rates and providing more reliable performance.

Ultimately, guardrails allow you to harness AI's benefits while mitigating risks, ensuring a more secure and trustworthy user experience.

# How does the OPEA architecture change?

Thanks to the interchangeability offered by OPEA, most of the components from the default [ChatQnA example](https://github.com/opea-project/GenAIExamples/tree/main/ChatQnA) (Module 1) remain unchanged, with the addition of a few that can be seamlessly integrated into a new deployment. The architecture closely resembles the ChatQnA setup without guardrails, with two additional components:

-   `chatqna-tgi-guardrails`
-   `chatqna-guardrails-usvc`
 
The `chatqna-tgi-guardrails` microservice launches a TGI server using the `meta-llama/Meta-Llama-Guard-2-8B` model. This model acts as a real-time filter to ensure that all queries processed through the ChatQnA pipeline adhere to defined safety protocols.
 
The `chatqna-guardrails-usvc` microservice serves as the operational endpoint analyzer (OPEA), evaluating **user queries** to determine if they request content deemed unsafe by the model.

## Deploying ChatQnA guardrails

Kubernetes enables the isolation of development environments through the use of multiple namespaces. Since the pods for the ChatQnA pipeline are currently deployed in the `default` namespace, you will deploy the guardrails in a separate namespace `guardrails`, to avoid any interruptions.

If you have been logged out of your CloudShell, click in the CloudShell window to restart the shell, or click the icon in the AWS Console to open a new CloudShell

1. Go to your CloudShell and deploy the ChatQnA-Guardrails ClourFormation template into your EKS Cluster

```bash
aws cloudformation execute-change-set --change-set-name guardrails-change-set --stack-name OpeaGuardrailsStack
```

{{% notice note %}}The manifest for *ChatQnA-Guardrails* can be found in the [ChatQnA GenAIExamples repository](https://github.com/opea-project/GenAIExamples/blob/main/ChatQnA/kubernetes/intel/cpu/xeon/manifest/chatqna-guardrails.yaml), and the instructions for deploying it manually can be found [here](https://github.com/opea-project/GenAIExamples/blob/main/ChatQnA/kubernetes/intel/README.md). The instructions you're using in this workshop use AWS CloudFormation templates created by the AWS Marketplace EKS package.
{{% /notice %}}

2. Verify the new namespace was created

```bash
kubectl get namespaces 
:::
You will see the `guardrails` namespace

![namespace](/images/namespace_guard.png)

3. Check pods on the `guardrails` namespace. It will take a few minutes for the models to download and for all the services to be up and running.

Run the following command to check if all the services are running:

:::code{showCopyAction=true language=bash}
kubectl get pods --namespace guardrails
```

![new_pods](/images/guardrails_pods.png)

Wait until the output shows that all the services including `chatqna-tgi-guardrails` and `chatqna-guardrails-usvc`  are running (1/1).

4. Verify the deployment is done verifying the new load balancer on your managment console

{{% notice note %}}Wait 5 minutes for the load balancer to be created.
{{% /notice %}}

![Alt tex](/images/lb_2.png)
![Alt text](/images/lb_1.png)
  
By checking that both the load balancer and pods are running, we can confirm that the deployment is ready and start testing the behavior.
 
