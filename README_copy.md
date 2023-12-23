
Your slides goes here...

### your Frist project using API gateway and lambda to control in instance EC2 in **AWS**

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![AWS](https://img.shields.io/badge/Python-14354C?style=for-the-badge&logo=python&logoColor=white)


in this article i will make an user friendly page to start, stop and check statue of the EC2 using **API gateway** and **lambda**  using **python**


1. create function From lambda -> functions 
    ![create function in lambda](https://github.com/MhmedRjb/photos/blob/main/1_Create_function.png?raw=true)
    - *Function name* a clear name 
    - *Architecture* x86_64 or arm64 work but according to AWS arm64 is lower in  cost
    - *Permissions* **Create a new role with basic Lambda permissions**
    basic Lambda permissions is permission to record **logs** which needed to add moer permission to stert/stop and check statue which will made later 

1. in function page there is code editaor with the default code and **test** for adding diffrent test **event** and **configration**  (Triggers,permissions,Environment variables,etc..)  ``` lambda_handler(event, context):```

    ###### **event**  is a JSON-formatted document that contains data for a Lambda function to process.

    ###### **context** This object provides methods and properties that provide information about the invocation, function, and runtime environment.

    ![function page](https://github.com/MhmedRjb/photos/blob/main/2_function.png?raw=true)

3. add the permissions for start/stop and decripe EC2 
    in configration → permissions → View the name-role-l0mmurec role  on the IAM console
    ![function page](https://github.com/MhmedRjb/photos/blob/main/permissions.png?raw=true)
    →then add permissions → create inline policy → 
    ![function page](https://github.com/MhmedRjb/photos/blob/main/inlinepermissions.png?raw=true)

    in this page Json  and use the right panal to add a diffrent things 
    ```json
    {
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [
				"ec2:StartInstances",
				"ec2:StopInstances",
				"ec2:DescribeInstances"
			],
			"Resource": [
				"*"
			]
		}
	]
    }
    ```
    - ```"Version": "2012-10-17"```: This is the policy language version; it defines the structure of the policy.
    - ```"Statement"```: This is an array of individual statements (policies). Each statement is a standalone policy.
    - ```"Sid": "Statement1"```: This is an optional identifier for the policy statement.
    - ```"Effect": "Allow"```: This defines the effect of the policy. In this case, it’s “Allow”, which means the actions specified in the policy are allowed.
    - ```"Action"```: This is a list of actions that the policy allows. Here, it allows starting instances ```ec2:StartInstances```, stopping instances ```ec2:StopInstances```, and describing instances ```ec2:DescribeInstances``` on EC2.
    - ```"Resource": ["*"]```: This defines the resources on which the actions can be performed. The "*" means all resources.

3. add the important variables to **Environment variables** in **configration** 
    ![add the important variables to **Environment variables** in **configration**  page](https://github.com/MhmedRjb/photos/blob/main/Environmentvariables.png?raw=true)

4. write the code 
    
    ```python 
    import boto3
    import logging
    import os 
    region = os.environ['region']
    instance_id = os.environ['instance_id']


    def lambda_handler(event, context): 
        try:
            ec2 = boto3.client('ec2', region_name=region)
            ec2.start_instances(InstanceIds=[instance_id], DryRun=False)
            # DryRun=False/true it used to check the permissions before excute or no check it

            #ec2.stop_instances(InstanceIds=[instance_id], DryRun=False)

            #response = ec2.describe_instances(InstanceIds=[instance_id])
            #instance_state = response['Reservations'][0]['Instances'][0]['State']['Name']
            return "Instance started"
        except Exception as e:
            raise  
     ```
     ##### if you got timed out error increes it in configuration → General configuration → edit ,make it 6 sec
  
    1. this demonstrate how the base code will be but before that we need to find  **condition to confirm** which  function will  excute start, stop or check state 

        this function use **APIgateaway** so what is the event APIgateaway send ,this could be find easly in **test event** 
        go to → test and choose as showen

        ![function page](https://github.com/MhmedRjb/photos/blob/main/testeventhttpapi.png?raw=true)

        use this part in json 
        ``` json
            "queryStringParameters": {
            "parameter1": "value1,value2",
            "parameter2": "value"
        }, 
        ```
        it looks like this in API endpoint 

        https://<api-id>.execute-api.<region>.amazonaws.com/<stage>/resource?**parameter1=value1,value2**&parameter2=value

        so for using this paramter ,it can ba accessed in python like this     
        ```python
        action = event['queryStringParameters']['parameter1']
        ```
        our lambda need only one parameter called ***action***

        https://.execute-api..amazonaws.com/default/?action=start
        https://.execute-api..amazonaws.com/default/?action=stop
        https://.execute-api..amazonaws.com/default/?action=state

        start by make three event by cahnge  apigateaway-http-proxy templet as shown before and give then names 

        ``` json
            "queryStringParameters": {
            "action": "start",
        }, 
        ```
        ``` json
            "queryStringParameters": {
            "action": "stop",
        }, 
        ```
        ``` json
            "queryStringParameters": {
            "action": "state",
        }, 
        ```

    2. now after make test event change the code for more complex one 
        This Python script is for managing AWS EC2 instances. It has three operations: ```state()```, ```start()```, and ```stop()```, which check the instance’s state, start the instance, and stop the instance, respectively. The ```lambda_handler()``` function is the entry point for the Lambda function, which uses a dictionary called ```function_map``` to map incoming actions to these operations (working like an **if statment** ) . This allows the script to call the right function based on the input ```action```. If an invalid action is received, it simply does nothing.



        1. **lambda_function.py**
        ```python
        import boto3
        import logging
        import os 
        from get_instance_state import state
        from start_instance import start_instances
        from stop_instance import stop_instance
        #get data from Environment variables
        region = os.environ['region']
        instance_id = os.environ['instance_id']

        def state():
            return get_instance_state(instance_id, region)

        def start():
            return start_instances(instance_id, region)

        def stop():
            return stop_instance(instance_id, region)
        # Lambda function entry point

        def lambda_handler(event, context):
            # Map the paths to the functions
            function_map = {
                "state": state,
                "start": start,
                "stop": stop
                }

            action = event['queryStringParameters']['action']
            # Use the get method to avoid KeyErrors
            function = function_map.get(action)
            if function:
                try:
                    return function()
                except Exception as e:
                    return {
                        'statusCode': 500,
                        'body': str(e)
                    }
            else:
                return {
                    'statusCode': 404,
                    'body': "404 Not Found - Invalid URL"
                }
        ```

        2. **start_instance.py**
        ```python 
        import boto3
        import logging
        import os 

        #get data from Environment variables
        region = os.environ['region']
        instance_id = os.environ['instance_id']
        # Lambda function entry point
        logger = logging.getLogger()
        logger.setLevel(logging.ERROR)
        ec2 = boto3.client('ec2', region_name=region)

        def start_instances(instance_id, region):
            try:
                ec2.start_instances(InstanceIds=[instance_id], DryRun=False)
                return "Instance started"
            except Exception as e:
                logger.error(f"Error starting instance: {str(e)}")
                raise
        ```
        2. **stop_instance.py**
        ```python
        import boto3
        import logging
        import os 
        #get data from Environment variables
        region = os.environ['region']
        instance_id = os.environ['instance_id']
        
        # Lambda function entry point
        logger = logging.getLogger()
        logger.setLevel(logging.ERROR)
        ec2 = boto3.client('ec2', region_name=region)

        def stop_instance(instance_id, region):
            try:
                ec2.stop_instances(InstanceIds=[instance_id], DryRun=False)
                return "Instance stopped"
            except Exception as e:
                logger.error(f"Error stopping instance: {str(e)}")
                raise

        ```
        3. **get_instance_state.py**
        ```python
        import boto3
        import logging
        import os 

        #get data from Environment variables
        region = os.environ['region']
        instance_id = os.environ['instance_id']

        # Lambda function entry point
        logger = logging.getLogger()
        logger.setLevel(logging.ERROR)
        ec2 = boto3.client('ec2', region_name=region)

        def get_instance_state(instance_id, region):
            try:
                response = ec2.describe_instances(InstanceIds=[instance_id])
                instance_state = response['Reservations'][0]['Instances'][0]['State']['Name']
                return instance_state
            except Exception as e:
                logger.error(f"Error getting instance state: {str(e)}")
                raise
        ```
        now you

7. now you need to add a trigger (APIgateaway) from diagram or **configuration** 
    ![](https://github.com/MhmedRjb/photos/blob/main/addtriger.png?raw=true)

    1. after add this trigger now **associated** it with the lambda
    go to the apigateaway 
    ![](https://github.com/MhmedRjb/photos/blob/main/endpoint.png?raw=true)
    ![Alt text](https://github.com/MhmedRjb/photos/blob/main/1.png?raw=true)
    ----

    ![Alt text](https://github.com/MhmedRjb/photos/blob/main/2.png?raw=true)


    ----
    ![](https://github.com/MhmedRjb/photos/blob/main/interactWithEC2-AP.png?raw=true)
    ![](https://github.com/MhmedRjb/photos/blob/main/Createanintegration1.png?raw=true)

    after intgrate it with the lambda now it is 
    ### ready to run 
    https://0****hm*y8.execute-api.us-east-1.amazonaws.com/default/?action=stop
    https://0****hm*y8.execute-api.us-east-1.amazonaws.com/default/?action=atate
    https://0****hm*y8.execute-api.us-east-1.amazonaws.com/default/?action=start
    
    


        








 

