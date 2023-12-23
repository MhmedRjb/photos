
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


 

