+++
title = "Deploy ChatQnA"
weight = 200
+++
## Introduction
In this section, you’ll deploy the OPEA blueprint for a RAG application (ChatQnA) on Amazon Elastic Kubernetes Service (EKS) environment.  This exploration will give you a better understanding of how the RAG application operates within a managed Kubernetes environment, allowing you to inspect the specific components and their roles in the system. You will have this configuration avaiable from [AWS Marketplace site](https://aws.amazon.com/marketplace/pp/prodview-yxrr7gseopq5e). ChatQnA is a RAG chatbot located in OPEA's GenAIExamples repository. 

[🔗 ChatQnA on GitHub](https://github.com/opea-project/GenAIExamples/tree/main/ChatQnA)

![OPEA Architecture](/images/opeaad.png)

Since you already set up your access to your Kubernetes clustes from "Getting Set Up" section, you will now explore and learn what you have deployed in your environment.

### Step 1: Examine Cluster Resources

1. Go to your CloudShell and deploy the ChatQnA ClourFormation template into your EKS Cluster

```bash
aws cloudformation execute-change-set --change-set-name default-change-set --stack-name OpeaChatQnAStack
```

### Step 2: Examine Cluster Resources

Click on the assigned cluster name to explore the deployment. Each EKS cluster includes essential configurations, such as the Kubernetes version, networking setup, and logging options. Review these settings in the console to gain insight into the foundation of your application environment. Understanding these details will help in managing cluster operations and troubleshooting if needed.
In this task we will focus on

1. Click on **Resources**

![Alt text](/images/resources.png)

This section in the console displays all the applications currently running within your cluster, including ChatQnA and any related microservices. Verify all the microservices of the OPEA ChatQnA blueprint are installed:

![OPEA Microservices](/images/services.png)

3. Get pod names (default namespace)

```bash
kubectl get pods
``` 

The output has to display all the pods "Running" (1/1)

```
NAME                                       READY   STATUS    RESTARTS   AGE
chatqna-65f6766db7-nlrr5                   1/1     Running   0          119s
chatqna-chatqna-ui-6f94b99d96-clvfw        1/1     Running   0          2m
chatqna-data-prep-59868bcc96-6d8jt         1/1     Running   0          2m
chatqna-nginx-67fc749576-4tzn4             1/1     Running   0          119s
chatqna-redis-vector-db-798f474769-5k4ct   1/1     Running   0          119s
chatqna-retriever-usvc-5dd6664b66-cs7rb    1/1     Running   0          119s
chatqna-tei-7f948c9bdd-8qdth               1/1     Running   0          119s
chatqna-teirerank-6c9cd6c546-lk2qk         1/1     Running   0          119s
chatqna-tgi-785cff64c4-k266l               1/1     Running   0          119s
```

This confirms that your application was succesfully deployed, and you’re now ready to explore your application deployment and manage resources within the cluster.
