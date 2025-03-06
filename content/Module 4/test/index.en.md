---
title : "Test the deployment and verify the RAG workflow"
weight : 202
---

## Understand RAG and use the UI
Now that you've verified all services are running, let’s take a look at the UI provided by the implementation.

To access the UI, open any browser and go to the DNS of the `ChatQnA Bedrock Load Balancer`: *http://bedrock-ingress-xxxxxxx.us-east-2.elb.amazonaws.com* (Modify with your `bedrock-ingress`DNS URL)

In the UI you can see the chatbot to interact with it

![ChatQnA_UI](/static/images/chatqna_ui.png)

Now when you send a prompt to the chatbot, the response will be coming from  Anthropic's Claude Haiku through Amazon Bedrock. 

