---
title : "Test the deployment"
weight : 201
---

### Test application

You can check the deployment by accessing to the DNS url of the load balancer the cloud formation templated created.

1. Look for the load balancer:

![lb](/static/images/lb.png)

2. Copy your DNS name for `bedrock-ingress`:

![Alt text](/static/images/br_ingress.png)

3. Paste it on a new browser tab to access to the interface 

In the UI you can see the chatbot to interact with it

![ChatQnA_UI](/static/images/chatqna_ui.png)

4. Check if the model is able to give us an answer about OPEA:

:::code{showCopyAction=true}
What is OPEA?
:::

![Alt text](/static/images/Bedrock_bad.png)

Notice that the initial answer provided by the chatbot is outdated or lacks specific information about OPEA. This is because OPEA is a relatively new project and wasn’t part of the dataset used to train the language model. Since most language models are static—they rely on data available at the time of training—they can’t automatically incorporate recent developments or information about new projects like OPEA.

5. Upload your context:

However, RAG offers a solution by enabling real-time context. Through the UI, you’ll see an icon that allows you to upload relevant context information. This action initiates a process where the document is sent to the **DataPrep** microservice to generate **embeddings**, and the data is then ingested into the **Vector Database**. 

By uploading a new document or link, you’re effectively expanding the chatbot’s knowledge base to include the latest information, which helps improve the relevance and accuracy of the responses.

![Attach Document](/static/images/Attach_document.png)

The deployment allows you to upload either a file or a site. For this case, use the OPEA site:

- Click on the **upload icon** to open the right panel
- Click on **Paste Link**
- Copy/paste the text `https://opea-project.github.io/latest/introduction/index.html` to the entry box
- Click **Confirm** to start the indexing process

When the indexing completes, you'll see an icon added below the text box, labeled **https://opea-project.github.io/latest/introduction/index.html**

![Document_uploaded](/static/images/Document_uploaded.png)

6. Ask the application after the context is provided:

Ask **"What is OPEA?"** again to see the updated answer. 

![Alt text](/static/images/what_is_opea.png)

This time, the chat bot responds correctly based on the data it added to the prompt from the new source, the OPEA web site. 

## Conclusion 

In this task, you tested the deployment of a RAG-powered chatbot using Bedrock. By uploading relevant context, the model learned and updated its responses based on the new information. This process demonstrated how the integration of RAG allows the system to improve its answers in real-time, leveraging Bedrock.


