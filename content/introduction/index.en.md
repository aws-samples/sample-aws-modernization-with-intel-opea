---
title : "Introduction"
weight : 10
---

## Introduction
In today’s fast-paced industry, staying up to date with the tools and best practices for building secure, reliable, and high-performance GenAI applications can be a real challenge. The rapid evolution of AI models, combined with the increasing complexity of deployment environments, means developers must constantly navigate new technologies, frameworks, and scalability concerns. Security, data integrity, and compliance standards add further pressure, making it critical to ensure applications not only perform well but also adhere to industry requirements.

For enterprises, the ability to implement AI solutions efficiently without sacrificing quality or time to market is essential for maintaining competitiveness. This demand for quick, scalable, and secure AI solutions requires a robust and streamlined approach, which is exactly where GenAI applications powered by frameworks like OPEA come in. By simplifying the deployment process and integrating best practices, OPEA helps developers overcome these challenges and deliver AI applications that meet enterprise demands. Before getting into the workshop, let’s cover the key technology you’ll use to create your RAG-based application.

## What is OPEA?
:link[OPEA (Open Platform for Enterprise AI)]{href=https://opea.dev external=false target="_blank"} is an open source project under the *LF & AI Data Foundation* that provides a framework for enabling the creation and evaluation of open, multi-vendor, robust, and composable generative AI (GenAI) solutions. OPEA simplifies the implementation of enterprise-grade composite GenAI solutions, starting with a focus on retrieval augmented generation (RAG). The platform is designed to facilitate the efficient integration and deployment of secure, performant, and cost-effective GenAI workflows into business systems , leading to quicker GenAI adoption and business value.OPEA is an open platform project that lets you create open, multi-vendor, robust, and composable GenAI solutions that harness the best innovation across the ecosystem.

## What is Retrieval Augmented Generation (RAG)?
RAG is a technique that combines two powerful AI capabilities: information retrieval and large language model (LLM) generation. Instead of relying solely on the LLM’s knowledge, RAG pulls in relevant, up-to-date information from external data sources, such as databases or documents, to augment the model's responses. This enhances accuracy and relevance and helps reduce hallucinations, especially for enterprise use cases where real-time or domain-specific knowledge is critical.

In this workshop, you'll leverage OPEA to build a RAG-based application that retrieves and processes external data, delivering more accurate and context-aware responses. This approach ensures your AI model not only generates human-like text but also incorporates the most relevant information to meet enterprise demands. The modular nature of OPEA allows you to easily integrate key components, such as applying safety guardrails and optimizing performance. By deploying RAG applications on AWS using OPEA, you'll experience firsthand how RAG can drive scalable, secure GenAI workflows tailored for enterprise use.

:button[More on RAG]{iconName="external" iconAlign="right" href="https://medium.com/p/4d1d08f736b3 "}

## The OPEA Project 
OPEA uses microservices to create high-quality GenAI applications for enterprises, simplifying the scaling and deployment process for production. These microservices leverage a service composer that assembles them into a megaservice thereby creating real-world enterprise AI applications.

It’s important to familiarize yourself with the MAIN key elements of OPEA:

1. [**GenAIComps**](https://github.com/opea-project/GenAIComps) 
A collection of microservice components that form a service-based toolkit. Each microservice is designed to perform a specific function or task within the GenAI application architecture. By breaking down the system into these smaller, self-contained services, microservices promote modularity, flexibility, and scalability. This modular approach allows developers to independently develop, deploy, and scale individual components of the application, making it easier to maintain and evolve over time. All of the microservices are containerized, allowing cloud native deployment. Here, you will find contributions to multiple partners/communities to further construction.

2. [**GenAIExamples**](https://github.com/opea-project/GenAIExamples)
While *GenAIComps* offers a range of microservices, *GenAIExamples* provides practical, deployable solutions to help users implement these services effectively. This repo provides use-case-based applications that demonstrate how the OPEA architecture can be leveraged to build and deploy real-world GenAI applications. In the repo, developers can find practical resources such as Docker Compose files and Kubernetes Helm charts, which help streamline the deployment and scaling of these applications. These resources allow users to quickly set up and run the examples in local or cloud environments, ensuring a seamless experience.

## AWS Services and Tools for the Lab

This lab leverages open source Kubernetes clusters due to their flexibility, scalability, and extensive ecosystem, which makes them ideal for running modern cloud-native applications. Kubernetes provides features like container orchestration, resource management, and scalability, which are essential for deploying and managing complex AI/ML pipelines such as GenAI deployments.

1. **Amazon Elastic Kubernetes Service (EKS)** : For AWS, we use `Amazon Elastic Kubernetes Service (EKS)`, a managed Kubernetes service, to simplify the setup and operation of Kubernetes clusters. EKS automates tasks like cluster provisioning, node management, and Kubernetes updates, allowing participants to focus on application development and deployment rather than underlying infrastructure.

2. **CloudFormation Templates** : The OPEA repository includes CloudFormation templates designed to deploy OPEA blueprints, which include the solutions and related infrastructure on AWS environments. These templates automate the provisioning of AWS resources like VPCs, IAM roles, and storage, ensuring a streamlined and consistent setup.

    The cloudformation templates are based on OPEA [`Helm charts/manifests`](https://github.com/opea-project/GenAIInfra/tree/main/helm-charts) that can be used on any Kubernetes deployments. These charts package Kubernetes resources (like deployments, services, and configurations) into reusable templates, allowing you to easily deploy and manage OPEA components in your Kubernetes cluster, particularly for the deployment of services and AI pipelines.To streamline the deployment process, all tasks in the lab are supported by preconfigured CloudFormation templates. These templates can be accessed from:

    -   The OPEA repository, where templates are curated for deploying RAG solutions and related infrastructure.
    -   The AWS Marketplace provides a convenient way to deploy OPEA solutions directly into your AWS environment. Go to [AWS Marketplace site](https://aws.amazon.com/marketplace/pp/prodview-yxrr7gseopq5e) to access to the templates.

## Be involved with OPEA
OPEA is an open source project that welcomes contributions in many forms. Whether you're interested in fixing bugs, adding new GenAI components, improving documentation, or sharing your unique use cases, there are numerous ways to get involved and make a meaningful impact. 

We’re excited to have you on board and look forward to the contributions you'll bring to the OPEA platform. 

For detailed instructions on how to contribute, please check out our [Contributing Documentation](https://opea-project.github.io/latest/community/CONTRIBUTING.html).

[Follow the project](https://github.com/opea-project) and stay tuned for new events!

#### Things To Remember

- This workshop uses accounts provided by AWS Workshop Studio and can only be used at AWS-run events
- This workshop uses M7i.24xl instance types. See [here](https://aws.amazon.com/ec2/pricing/on-demand/) for cost information. 
- This workshop runs in the :param{key=region} :param{key=regionName} region, but you can perform these activities in any region that has proper EC2 capacity.
- You are not obligated to provide any personal information as a condition of participating in this workshop

#### Contact

this course is intended to be a live guide for OPEA on AWS. If you face errors when deploying, please let us know:

Ezequiel Lanza : ezequiel.lanza@intel.com
