---
title : "Integrate Inference API (Denvr Cloud and Intel Gaudi AI Accelerator)"
weight : 200
---

# When do you need to use managed Inference APIs?

As enterprise-level RAG deployments grow more complex, migrating certain components to managed services becomes increasingly advantageous. This approach reduces deployment complexity, simplifies infrastructure management, and addresses the need for auto-scaling. Additionally, it provides developers with an easier and faster pathway to access LLMs (Large Language Models).

# How was Denvr Cloud Inference API enabled?

The OPEA LLM and Embedding microservices are deployed on Intel Gaudi2 AI Accelerators, hosted by Denvr Cloud. These APIs are secured with OPEA authentication services and load-balanced for optimal performance. While the initial implementation was powered by OPEA, Denvr Cloud further fine-tuned and optimized the setup to meet specific requirements.

Refer the architecture for deployment information:

![Architecture](/static/images/arch.png)

# How does the OPEA architecture change?

Thanks to the interchangeability offered by OPEA, most of the components from the default [ChatQnA example](https://github.com/opea-project/GenAIExamples/tree/main/ChatQnA) (Module 1), with the additional component that can be seamlessly integrated into a new deployment. The architecture closely resembles the ChatQnA setup without guardrails, with an additional component:

- `chatqna-llm-uservice`

The `chatqna-llm-uservice`` serves as wrapper service for Inference API and aligns with vLLM or OpenAI compatible endpoints.This service connects with the Inference APIs and provides necessary configurations.

## Deploying Chat QnA with Inference API

For this lab, we've created a changeset with the full parallel deployment of the ChatQnA example in the same Kubernetes cluster you've been using. The following command will deploy pods to the cluster within the "remote-inference" namespace that are identical to the original ChatQnA pods, except with LLM remote inference models instead of TGI.

1. Get your keys. *Tokens have to be provided by Denvr Cloud. These API tokens are enabled only for private preview for the workshop duration. For getting access to the inference API's, please contact Denvr Cloud (vaishali@denvrdata.com)*.

2. Deploy the ChatQnA using Denvr Cloud Inference API (Meta Llama 70B 3.1 Instruct) into your EKS Cluster. 

    This will deploy Chat QnA with Meta-Llama -3.1-70B-instruct Inference API. These Inference API's are pre-deployed on Denvr Cloud with Intel Gaudi2 accelerator and OPEA LLM microservices.

**Note** : REPLACE DenvrClientID & DenvrClientSecret for the values you previously got from Denvr.

:::code{showCopyAction=true}
aws cloudformation execute-change-set --change-set-name remote-inference-change-set --stack-name OpeaRemoteInferenceStack 
:::

:::alert
*The manifest for *ChatQnA-Remote Inference* can be found in the [ChatQnA GenAIExamples repository](https://github.com/opea-project/GenAIExamples/blob/main/ChatQnA/kubernetes/intel/cpu/xeon/manifest/chatqna-remote-inference.yaml), and the instructions for deploying it manually can be found [here](https://github.com/opea-project/GenAIExamples/tree/main/ChatQnA/kubernetes/intel#deploy-on-xeon-with-remote-llm-model). The instructions here use AWS CloudFormation templates created by the AWS Marketplace EKS package*.
:::

2. Verify the new services are created

:::code{showCopyAction=true}
kubectl get pods -n remote-inference
:::

![namespace](/static/images/pods.png)

3. Test the Chat QnA on the console
    
    Access to ngnix POD (copy your NGNIX pod name from kubectl get pods -n remote-inference and REPLACE *chatqna-nginx-xxxxxxxx* on the below command)

    :::code{showCopyAction=true}
    kubectl exec -it <*POD name:chatqna-nginx-xxxxxxxx*> --namespace=remote-inference -- /bin/bash
    :::

    Your command prompt should now indicate that you are inside the container, reflecting the change in environment:

    :::code{showCopyAction=false}
    root@chatqna-nginx-deployment-xxxxxxxxxxxx:/#
    :::

    Get the "What is Deep Learning? Explain in 20 words"*:

    :::code{showCopyAction=true language=json}
    curl chatqna:8888/v1/chatqna -H 'Content-Type: application/json' -d '{"messages": "What is Deep Learning. Exaplain in 20 words?"}'
    :::

    :::code{showCopyAction=false}
    data: b' \n'

    data: b'Deep'

    data: b' learning'

    data: b' is'

    data: b' a'

    data: b' subset'

    data: b' of'

    data: b' machine'

    data: b' learning'

    data: b' that'

    data: b' uses'

    data: b' neural'

    data: b' networks'

    data: b' to'

    data: b' analyze'

    data: b' data'

    data: b' and'

    data: b' make'

    data: b' predictions'

    data: b'.'

    data: b''

    data: [DONE]
    :::
    
4. Verify the deployment is done verifying the new load balancer on your managment console

    ![Alt tex](/static/images/lb_1_denvr.png)

    ![Alt text](/static/images/lb_2_denvr.png)
  
We can confirm the deployment is ready and we could start testing the behaviour.