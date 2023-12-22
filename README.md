
Your slides goes here...

### your Frist project using API gateway and lambda to control in instance EC2 in **AWS**
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![AWS](https://img.shields.io/badge/Python-14354C?style=for-the-badge&logo=python&logoColor=white)


in this article i will make an user friendly page to start, stop and check statue of the EC2 using **
API gateway** and **lambda**  using **python**


1. we start by create function in lambda page and choose as shown below
![create function in lambda](https://github.com/MhmedRjb/photos/blob/fc3a66975ca80c545a626b1aa8e5ebed22632271/Untitled.jpg?raw=true)  

2. in lambda page you have a code area and diagram/JSON describe the function the implementation
click in **add trigger** in diagram area I will use HTTP for this mission ,make secuirty open for your frist one 

![create function in lambda](https://github.com/MhmedRjb/photos/blob/main/addtriger.png?raw=true)


4. *configration* then *triggers* you will see the **endpoint api** and test it in browser you will see "Hello from Lambda!" which exist in defult code 


 ![Alt text](https://raw.githubusercontent.com/MhmedRjb/photos/b79afb3a4e2dbc3eb69c678976aa5536a38a97c9/APIendpoint.png)

 5. there is another way to test the code without  **endpoint api** by *test* in code tabe after make **test event** with the ability to add another custome test even writen by you or  a commen test event we will go to it later 
 ![Alt text](https://github.com/MhmedRjb/photos/blob/main/testevent.png?raw=true)

 1. before starting to write code  we need to assign a **role** have **policies** with **permissions** to (start, stop ,checking staue of EC2 ,etc..)
    1. frist making the policy which allowed us to do this  
    by going to IAM mangment then **Policies** then **Create policy**  
    - by **json** you can write the permissions using the right panal to indify the **Action**,**Resource** as in this code 

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
                        "*",
                        "arn:aws:ec2:us-east-1:1234566789:instance/i-12wq2f45w6c123456" 
                    ]
                }
            ]
        }
        ```

        **Statement**: This is an array of individual statements. Each statement is a standalone permission.

        **Sid**: *Statement1*: This is an optional identifier for the statement.
        **Effect**: *Allow* or *deny*: This defines the effect of the statement. 
        **Action**: This is a list of actions that the policy allows. In this case, it allows ***starting*** and ***stopping*** EC2 instances, and ***describing the status*** of an instance.
        **Resource**: This is a list of resources to which the actions can apply. In this case, it applies to ***all resources** ("*") and ***a specific EC2 instance*** 
        ("arn:aws:ec2:us-east-1:1234566789:instance/i-12wq2f45w6c123456").

        then give it a name and save it 
    2. after making this Policy assign it to the role in frist step (creat function) i choose  
    - create a new rolw with baisc lambda permissions (permissions to record logs)
     ![Alt text](https://github.com/MhmedRjb/photos/blob/main/Screenshot%202023-12-22%20175433.png?raw=true)
    - go to role page from roles in IAM mangment or configration tab and 
**Add permissions** to it ,search for the name you chose and add it  
     ![Alt text](https://github.com/MhmedRjb/photos/blob/main/rolespage.png?raw=true)
     as you notice  there is a Policy name added which is baisc lambda permissions

     
 1. now returen to your function page 
    1. in code area add this code which for describe_instances and test it 
```python
import boto3

region = 'us-east-1'
instance_id = ['i-01fe23db04b579488']

def lambda_handler(event, context):
    ec2 = boto3.client('ec2', region_name=region)
    response = ec2.describe_instances(InstanceIds=instance_id)
    instance_state = response['Reservations'][0]['Instances'][0]['State']['Name']
    return instance_state
 ```

8. in code area add this code which for describe_instances and test it by **end point API** or **test event** 
if this code work fine now we can increes the complexity of it by addind stop and start 




