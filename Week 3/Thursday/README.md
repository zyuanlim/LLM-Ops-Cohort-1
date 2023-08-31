# Building A Finetuning Machine Using Chainlit

Today we'll take a look at how we can wrap our finetuning process into Chainlit and have our own dynamic fine-tuning machine. 

We'll be leveraging [this](https://github.com/AI-Maker-Space/LLM-Ops-Vault/blob/main/Week%202/Second%20Session/Automated%20Fine-tuning%20with%20LLamaIndex.ipynb) notebook for finetuning.

# Build 🏗️
There are 2 main tasks for this assignment:

- Verify you can run the notebook
- Wrap the notebook using Chainlit

# Ship 🚢
Construct a Chainlit application using the notebook that allows users to interface with the application.

# Share 🚀
Make a social media post about your final application and tag @AIMakerspace

# How to Deploy Your Own Sagemaker Endpoint in AWS

### Deploy Model

First things first, you'll need to make sure you have an AWS account, and that you have access to at least 1 `ml.g5.2xlarge` instances.

> You can check/request available quota through `ServiceQuota`

Next, we'll want to navigate to `Amazon SageMaker`, and create a domain! You can create a default domain - though you may have to ensure you have the correct IAM permission if you're not the admin on the account.

After you've done that, you'll want to open SageMaker Studio and navigate to the `Featured JumpStart models` section:

![image](https://i.imgur.com/eLZFfMM.png)

Now, we'll want to deploy our model to an endpoint using the `Deploy` button.

![image](https://i.imgur.com/0dxtAkX.png)

This process will take some time. Please be patient!

### Create Lambda with API Gateway

Next, we'll head to Lambda and `Create Function`!

We'll want to ensure that we create the function with a new role that we can edit to include the `InvokeEndpoint` permissions.

Our Lambda will have the following code: 

```python
import os
import boto3
import json

# grab environment variables
ENDPOINT_NAME = os.environ['ENDPOINT_NAME']
runtime= boto3.client('runtime.sagemaker')

def lambda_handler(event, context):
    response = runtime.invoke_endpoint(EndpointName=ENDPOINT_NAME,
                                       ContentType='application/json',
                                       Body=event['body'],
                                       CustomAttributes="accept_eula=true")
    
    result = json.loads(response['Body'].read().decode())
    
    
    return {
        "statusCode": 200,
        "body": json.dumps(result)
    }
```

And we'll want to add an environment variable by the name `ENDPOINT_NAME`  with our SageMaker endpoint name to our Lambda as well!

Now, we can add the `API Gateway`, which can be done by adding a `Trigger` in the Function overview. We will want it set to `Open` for ease of use.

Let's add the required permissions to our role once we add our Lambda!

![image](https://i.imgur.com/3ertSKe.png)

![image](https://i.imgur.com/GvGSLl6.png)

We'll add it as an `Inline Policy`:

```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": "sagemaker:InvokeEndpoint",
			"Resource": "*"
		}
	]
}
```

After this, we should be good to query our endpoint and carry on to the Notebook!

Make sure to add your API Gateway URL!