---
title: Agent to Agent Calls in Amazon Connect
---
The AWS Knowledge Center recently published an [article](https://aws.amazon.com/premiumsupport/knowledge-center/connect-agent-to-agent-extensions/) on how to set up agent to agent calling in Amazon Connect. I created an Amazon Connect instance to try it out and learn more about the service.  The article got me most of the way there but I'll go into detail on anything that I found confusing or I believe could have been explained better. 
### Create an Amazon Connect Instance
If you don't already have an Amazon Connect instance created, navigate to the service from the management console. Select **Add an instance** and the instance creation wizard  will start. The first step is to choose the method of identity management. I went with **Store users within Amazon Connect**. Step 2, create an admin. Step 3, check the boxes for incoming and outbound and select next until you create the instance.
### Create a DynamoDB Table
Follow the instructions in the article. The table must be named **AgenttoAgent** unless the table name is changed in the Lambda Function we will create shortly . When adding items to the table, the field for the agent's login name should be named exactly what the variable is named in the later Lambda Function. The items in your table should look something like this. Make sure to take note of the table's ARN which can be found in the **Overview** tab
![image](/assets/img/DBTableItem.png)
### Create an IAM Role
Create a role from the IAM Console and when it prompts you to select a policy, select **Create policy**. Select the **JSON** tab and paste the JSON policy. 
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:BatchGetItem",
                "dynamodb:GetItem",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:BatchWriteItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem"
            ],
            "Resource": "Replace with ARN of DynamoDB table you created"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```
Make sure to change the value of the first ```Resource``` key with the ARN of the DynamoDB table.
### Create a Lambda Function
Create a new Lambda Function from the Lambda Console. Choose a Python runtime above 2.7 as it will soon no longer be supported. Select the **Change default execution role**
 drop down. Select the **Use an existing role drop down** and select the IAM role that we just created. Create the function, open **lambda_function.py** in the code editor and replace the existing code with the code block from the article and **Deploy**.
### Add Lambda Function to Amazon Connect
Go to the Amazon Connect Console. Select your instance. Select the **Contact Flows** tab on the left and under **AWS Lambda** select your function and Add

![image](/assets/img/ConnectLambdaAdd.png)
### Create Call Flows and Quick Connect
The instructions to create the call flows and the quick connect are pretty straight forward. Here's what mine looked like ![image](/assets/img/queueFlow_2021-07-05_194737.png)
![image](/assets/img/ContactFlow.png)
### Test
I created two users and added an extension for one of them. I also set the one with an extension to use a "deskphone" from their Contact Control Panel so I could forward the call to my cell. 
![image](/assets/img/extensionDial_2021-07-05_202014.png)
![image](/assets/img/extensionDialConnected2021-07-05_202051.png)
### Conclusion
I was suspicious that because this works by dialing the number associated to the inbound queue we created earlier that users without the quick connect could still use the feature by dialing the number directly. I switched my user back to softphone, called the associated number, and was still able to connect to the user by extension.

![image](/assets/img/ManualConnect_2021-07-05_205638.png)
