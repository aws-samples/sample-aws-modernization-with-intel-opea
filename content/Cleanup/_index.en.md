+++
title = "IMPORTANT:How to Clean Up Your Environment"
weight = 1000
+++
 
 
## Overview

After completing the workshop, or if you encounter any deployment failures, you should clean up your AWS resources to prevent unnecessary charges. This guide provides step-by-step instructions for properly removing all deployed resources.

## Clean-up Process

Follow these steps in the specified order to ensure all resources are properly deleted:

### Step 1: Access AWS CloudFormation Console
Navigate to the AWS CloudFormation console in your AWS Management Console.

![CloudFormation Console](/images/cfn_clean.png)

### Step 2: Delete the Stacks in the Following Order
For a proper cleanup, you must delete the stacks in this specific sequence as shown in the image below:

![CloudFormation Stack Deletion Order](/images/Cleanup.png)

1. **First, delete the OPEA blueprint stacks**
   * Select and delete any child stacks that were created during deployment (such as `OpeaChatQnAStack`)
   * Wait for each deletion to complete before proceeding to the next stack
   * This ensures resources with dependencies are removed first and prevents deletion failures

2. **Delete the `OpeaEksStack`**
   * This removes the Kubernetes cluster and associated resources
   * Note: You don't need to manually delete the nested stacks, as they will be automatically removed when this parent stack is deleted

3. **Delete the `CDKToolkit` stack**
   * This removes the AWS CDK deployment infrastructure 

4. **Finally, delete the `LaunchStack`**
   * This removes the initial resources that were used to bootstrap the environment

## Troubleshooting Deletion Issues

### Common Issues and Solutions

1. **Stack Deletion Failures**
   * If a stack fails to delete normally, you can use the "Force Delete" option in CloudFormation
   * To do this, select the stack, click the "Delete" button, and check the "Force Delete" option when prompted
   * Note that forced deletion might leave some resources undeleted which may need manual cleanup

2. **S3 Bucket Deletion Issues**
   * A common issue is that S3 buckets containing workshop images aren't automatically removed
   * If this happens:
     * Navigate to the S3 service in your AWS Console
     * Look for the bucket startin with `cdk....`.
     * Empty the bucket (delete all objects) before attempting to delete the bucket itself

![CDK Stack in CloudFormation](/images/cdk_screen.png)

## Final Verification

After completing all deletion steps:
1. Refresh your CloudFormation console and verify no workshop-related stacks remain
2. Check the following services to ensure all resources are removed:
   * EKS (Kubernetes clusters)
   * EC2 (instances, load balancers, security groups)
   * S3 (storage buckets)
   * IAM (roles and policies created for the workshop)

This verification step is crucial to prevent unexpected charges on your AWS account.


