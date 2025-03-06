+++
title = "Verify OPEA Chat QnA with Inferenece API"
weight = 201
+++
## Test with ChatQnA UI

Now that you've verified all services are running, let’s take a look at the UI provided by the implementation.

To access the UI, open any browser and go to the DNS of the ChatQnA Load Balancer: http://denvr-ingress-xxxxxxx.us-east-2.elb.amazonaws.com  (Modify with your denvr-ingress DNS URL)

In the UI you can see the chatbot interact with it

![Alt text](/images/ChatQnA_Screen.png)

To verify the UI, go ahead and ask

    ```bash
What is Denvr? Provide short answer?
```

![Alt text](/images/question1.png)
    
Notice that the initial answer provided by the chatbot is outdated or generic or lacks specific information about Denvr Cloud. This is because Denvr Cloud wasn’t part of the dataset used to train the language model. Since most language models are static—they rely on data available at the time of training.

However, through the UI, you’ll see an icon that allows you to upload relevant context information. This action initiates a process where the document is sent to the DataPrep microservice to generate embeddings, and the data is then ingested into the Vector Database.

By uploading a new document or link, you’re effectively expanding the chatbot’s knowledge base to include the latest information, which helps improve the relevance and accuracy of the responses.

![Alt text](/images/Upload.png)

The deployment allows you to upload either a file or a site. For this case, use the Denvr Datawork's site:

- Click on the upload icon to open the right panel
- Click on Paste Link
- Copy/paste the text https://www.denvrdata.com/intel to the entry box
- Click Confirm to start the indexing process

When the indexing completes, you'll see an icon added below the text box, labeled https://www.denvrdata.com/intel

![Alt text](/images/context.png)

Ask "What is Denvr?" again to see the updated answer.

![Alt text](/images/question2.png)

This time, the chatbot responds correctly based on the data it added to the prompt from the new source, the Denvr Cloud website.

## Conclusion

In this workshop, participants learned how to integrate managed Inference APIs. This session showcase that OPEA is a flexible framework to deploy a pipeline across multiple cloud provders (AWS and Denvr Cloud) and using OPEA microservices to deploy LLMs in Intel Gaudi2 AI Accelerators.