
# GenAI ChatBot with Amazon Bedrock Agent

## Table of Contents
- [Introduction](#Introduction)
- [Prerequisites](#Prerequisites)
- [Target technology stack](#Target-technology-stack)
- [Deployment](#Deployment)
- [Useful CDK commands](#Useful-CDK-commands)
- [Code Structure](#Code-Structure)
- [Customize the chatbot with your own data](#Customize-the-chatbot-with-your-own-data)


## Introduction

This demo ChatBot application is developed using Amazon Bedrock including KnowledgeBase, Agent and other AWS's serveless GenAI solution. The solution demonstrates a Chatbot that utilizes the knowledge of EC2 instances and EC2 instance pricing. This Chatbot serves as an example of the power of Amazon Bedrock in processing and utilizing complex data sets, and it’s capability of converting natural language into Amazon Athena queries. It employs open source tools like LLamaIndex to enhance its data processing and retrieval capabilities. The solution also highlights the integration of various AWS resources, including Amazon S3 for storage, Amazon Bedrock KnowledgeBase to support the retrieval augmented generation (RAG), Amazon Bedrock agent to execute multistep tasks across data sources, AWS Glue for data preparation, Amazon Athena for efficient querying, Amazon Lambda for serverless computing, and Amazon ECS for container management.  These resources collectively enable the Chatbot to effectively retrieve and manage content from documents and databases, illustrating the potential of Amazon Bedrock in sophisticated Chatbot applications.

## Prerequisites
 * Docker
 * AWS CDK Toolkit 2.114.1+, installed installed and configured. For more information, see Getting started with the AWS CDK in the AWS CDK documentation.
 * Python 3.11+, installed and configured. For more information, see Beginners Guide/Download in the Python documentation.
 * An active AWS account
 * An AWS account bootstrapped by using AWS CDK in us-east-1 or us-west-2. Enable Claude model access in Bedrock service.

## Target technology stack
 * Amazon Bedrock
 * Amazon OpenSearch Serverless
 * Amazon ECS
 * AWS Glue
 * AWS Lambda
 * Amazon S3
 * Amazon Athena
 * Elastic Load Balancer

## Deployment
To run locally, first add a .env file to 'code/ecs-app/streamlit-app' folder containing the following

```.env
ACCOUNT_ID = <Your account ID>
AWS_REGION = <Your region>
LAMBDA_FUNCTION_NAME =  invokeAgentLambda # Sets name of choice for the lambda function called by streamlit for a response. Currently invokes an agent. 
```
Currenly, the code is working on us-east-2. 

This is a blank project for CDK development with Python.

The `cdk.json` file tells the CDK Toolkit how to execute your app.

This project is set up like a standard Python project.  The initialization
process also creates a virtualenv within this project, stored under the `.venv`
directory.  To create the virtualenv it assumes that there is a `python3`
(or `python` for Windows) executable in your path with access to the `venv`
package. If for any reason the automatic creation of the virtualenv fails,
you can create the virtualenv manually.

To manually create a virtualenv on MacOS and Linux:

```
$ python3 -m venv .venv
```

After the init process completes and the virtualenv is created, you can use the following
step to activate your virtualenv.

```
$ source .venv/bin/activate
```

If you are a Windows platform, you would activate the virtualenv like this:

```
% .venv\Scripts\activate.bat
```

Once the virtualenv is activated, you can install the required dependencies.

```
$ pip install -r requirements.txt
```
To add additional dependencies, for example other CDK libraries, just add
them to your `setup.py` file and rerun the `pip install -r requirements.txt`
command.

At this point you can now synthesize the CloudFormation template for this code.

```
$ cdk synth
```

To add additional dependencies, for example other CDK libraries, just add
them to your `setup.py` file and rerun the `pip install -r requirements.txt`
command.

You will need to bootstrap it if this is your first time running cdk at a particular account and region.

```
$ cdk bootstrap
```
Once it's bootstrapped, you can proceed to deploy cdk.

```
$ cdk deploy
```
If this is your first time deploying it, the process may take approximately 30-45 minutes to build several Docker images in ECS (Amazon Elastic Container Service). Please be patient until it's completed. Afterward, it will start deploying the chatbot-stack, which typically takes about 5-8 minutes.

Once the deployment process is complete, you will see the output of the cdk in the terminal, and you can also verify the status in your CloudFormation console.

You can either test the agent in AWS console or through streamlit app url listed in the outputs of chatbot-stack in CloudFormation.

To delete the cdk once you have finished using it to avoid future costs, you can either delete it through the console or execute the following command in the terminal.
```
$ cdk destroy
```
You may also need to manually delete the S3 bucket generated by the cdk. Please ensure to delete all the generated resources to avoid incurring costs.

## Useful CDK commands

 * `cdk ls`          list all stacks in the app
 * `cdk synth`       emits the synthesized CloudFormation template
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk docs`        open CDK documentation
 * `cdk destroy`     dstroys one or more specified stacks

## High-level Code Structure
```
code                              # Root folder for code for this solution
├── lambdas                           # Root folder for all lambda functions
│   ├── action-lambda                     # Lambda function that acts as an action for the Amazon Bedrock Agent
│   ├── create-index-lambda               # Lambda function that create Amazon Opensearch serverless index as Amazon Bedrock Knowlege base's vector database
│   ├── invoke-lambda                     # Lambda function that invokes Amazon Bedrock Agent, which is called diretly from the streamlit app
│   └── update-lambda                     # Lambda function that update/delete resources after AWS resources deployed via AWS CDK.
├── layers                            # Root folder for all lambda layers
│   ├── boto3_layer                       # Boto3 layer that is shared across all lambdas
│   └── opensearch_layer                  # opensearh layer that installs all dependencies for create Amazon Opensearch serverless index.
├── streamlit-app                         # Steamlit app that interacts with the Amazon Bedrock Agent
└── code_stack.py                     # Amazon CDK stack that deploys all AWS resources
```

## Customize the chatbot with your own data

To integrate your custom data for deploying the solution, please follow these structured guidelines tailored to your requirements:
 
### For Knowledgebase Data Integration:

#### 1. Data Preparation:

* Locate the `assets/knowledgebase_data_source/` directory.
* Place your dataset within this folder.

#### 2. Configuration Adjustments:

* Access the `cdk.json` file.
* Navigate to the `context/configure/paths/knowledgebase_file_name` field and update it accordingly.
* Further, modify the `bedrock_instructions/knowledgebase_instruction` field in the `cdk.json` file to accurately reflect the nuances and context of your new dataset.

### For Structural Data Integration:

#### 1. Data Organization:

* Within the `assets/data_query_data_source/` directory, create a subdirectory, for example, tabular_data.
* Deposit your structured dataset (acceptable formats include **CSV**, **JSON**, **ORC**, and **Parquet**) into this newly created subfolder.
* If you are connecting to your **existing database**, update the function `create_sql_engine()` in `code/lambda/action-lambda/build_query_engine.py` to connect to your database.

#### 2. Configuration and Code Updates:

* Update the `cdk.json` file's `context/configure/paths/athena_table_data_prefix` field to align with the new data path.
* Revise `code/lambda/action-lambda/dynamic_examples.csv` by incorporating new text to SQL examples that correspond with your dataset.
* Revise `code/lambda/action-lambda/prompt_templates.py` to mirror the attributes of your new tabular data.
* Modify the `cdk.json` file's `context/configure/bedrock_instructions/action_group_description` field to elucidate the purpose and functionality of the action lambda tailored for your dataset.
* Reflect the new functionalities of your action lambda in the `assets/agent_api_schema/artifacts_schema.json` file.

### General Update:

* In the `cdk.json` file, under the `context/configure/bedrock_instructions/agent_instruction section`, provide a comprehensive description of the Amazon Bedrock Agent's intended functionality and design purpose, taking into account the newly integrated data.

These steps are designed to ensure a seamless and efficient integration process, enabling you to deploy the solution effectively with your bespoke data.

Enjoy!