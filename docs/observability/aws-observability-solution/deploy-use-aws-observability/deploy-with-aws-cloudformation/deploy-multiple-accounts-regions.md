---
id: deploy-multiple-accounts-regions
---

# Deploy to Multiple Accounts and Regions

This topic has instructions for deploying AWS Observability Solution to multiple AWS accounts and regions  
using [Stack Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-concepts.html).

Given that we use an account alias, we recommend you use StackSets to automatically deploy the AWS CloudFormation template across all regions in one AWS account at a time. New AWS accounts can be added to an existing stack set.

## Before you start

You need to install apps and determine account aliases before deploying.

## Step 1: Install Apps

1. Complete the prerequisites for StackSets as described in the [AWS documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-prereqs.html).
1. Install the apps by running the AWS CloudFormation Stack once in any given account and region. Use the configuration below to set up only app dashboards.
1. Install AWS Observability Apps as **Yes**.  
    ![Multiaccount 1.png](/img/observability/Multiaccount_1.png)
1. Select the Sumo Logic Metrics Sources to create as **None**.  
    ![Multiaccount 2.png](/img/observability/Multiaccount_2.png)
1. Enable ALB Access logging as **None** and Create Sumo Logic ALB Logs Source as **No**.  
    ![Multiaccount 3.png](/img/observability/Multiaccount_3.png)
1. Create Sumo Logic CloudTrail Logs Source as **No**.  
    ![Multiaccount4.png](/img/observability/Multiaccount_4.png)
1. Create Sumo Logic CloudWatch Logs Source as **No**.  
    ![Multiaccount5.png](/img/observability/Multiaccount_5.png)
1. Enable ELB Classic Access logging as **None** and Create Sumo Logic ELB Classic Logs Source as **No**.  
    ![Multiaccount6.png](/img/observability/Multiaccount6.png)
1. Location where you want the App to be Installed as **PersonalFolder**. And for **Do you want to share App with whole organization**, set as **True**.  
    ![Multiaccount7.png](/img/observability/Multiaccount7.png)

## Step 2: Determine Account Aliases

If you are going to deploy the solution in multiple AWS accounts, we highly recommend that you prepare a CSV file that maps your AWS Account-ids to account aliases. These aliases should be something that makes it easy for you to identify what this AWS account is being used for (for example dev, prod, billing, and marketplace). These names will appear in the Sumo Logic Explorer View, metrics, and logs and can be queried using the “account field”.

The following is an example of the CSV file format to use:

`accountid,alias`

`234234234324,dev`

`214324324324,prod`

Please upload this file to an Amazon S3 bucket and make it accessible to the account from where you are going to run the CloudFormation template. 

In case you do not provide a CSV file or if we detect that it does not have the right format, the AWS account-id will be used as the alias and this value will be used for the “account” field in Sumo Logic.

### Use the AWS CloudFormation template with StackSets

1. Go to [Stack Sets](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacksets)
    in your AWS account.
1. Click **Create StackSet**.  
    ![ClodFormation_Stackset 1.png](/img/observability/ClodFormation_Stackset_1.png)
1. Paste the URL - https://sumologic-appdev-aws-sam-apps.s3.amazonaws.com/aws-observability-versions/v2.4.0/sumologic_observability.master.template.yaml in the Amazon S3 URL option and click **Next**.
    
    If you would like to download or inspect this or other versions of this template, please visit the [change log](../aws-observability-change-log.md) page.

    ![multi-create-stack.png](/img/observability/multi-create-stack.png)

1. Provide a StackSet Name and supply the values for each of the prompts listed as per instructions in the [Deploy the AWS Observability Solution](/docs/observability/aws-observability-solution/deploy-use-aws-observability) section with the following exception:
    * Leave the field “Alias for AWS Account Identification” blank.  
        
        ![aws-field.png](/img/observability/aws-field.png)

    * Provide the S3 Object URL of a CSV file that maps AWS Account IDs to an Account Alias in Section 2 of the template “AWS Account Alias”.  
        
        ![aws-url.png](/img/observability/aws-url.png)

    * Answer **No** in Section 3 of the template "Install AWS Observability Apps".   
        
        ![ClodFormation_Stackset 3.png](/img/observability/ClodFormation_Stackset_3.png)

    * Click **Next**.
1. Add Tags, select the Administrator role defined in the prerequisites above, and click **Next**.

    ![ClodFormation_Stackset 4.png](/img/observability/ClodFormation_Stackset_4.png)

1. Provide a single AWS account number only and select a list of regions in the account where you would like to deploy the AWS CloudFormation template as shown in the screenshot below:

    * You will need to select all the regions in the current account where you would like to deploy the template.  
        
        ![ClodFormation_Stackset 5.png](/img/observability/ClodFormation_Stackset_5.png)

1. Increasing the **Maximum concurrent actions** to be more than 1 is not recommended and can cause your stack set deployment to fail. Stack sets should be deployed one at a time, sequentially. Click **Next**.  
    
    ![ClodFormation_Stackset  6.png](/img/observability/ClodFormation_Stackset_6.png)

1. Review the details, select the capabilities and click **Submit**.  
    
    ![ClodFormation_Stackset 7.png](/img/observability/ClodFormation_Stackset_7.png)

1. Once you hit submit, the AWS CloudFormation template will execute in the provided account and regions sequentially.

## Add more accounts to the same Stack Set

1. Select **Add new stacks to StackSet**.

    ![ClodFormation_Stackset 8.png](/img/observability/ClodFormation_Stackset_8.png)

1. In ‘**Set deployment options**’, Enter the account number and regions you want to deploy the stack.  
    
    ![ClodFormation_Stackset 9.png](/img/observability/ClodFormation_Stackset_9.png)

1. Verify the S3 mapping file contains the mapping for the new Account ID.
1. Review the details, select the capabilities, and click **Submit**.