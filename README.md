# To-do list on aws 

## Using Various services of AWS 
- S3 bucket
- DynamoDB
- API Gateway
- IAM
- Lambda

## Web application for to-do list on aws
### Steps:
1. Create Table (DynamoDB)
- Search DynamoDB
- Create Table:
   - Table name: Tasks
   - Partition key: TaskID
   - Create Table
- Goto created Table Tasks:
   - Actions --> Create Items:
     
                 a. TaskID (PK) = 101
                 b. Add new attribute --> String --> TaskName = ITR Filling
                 c. Add new attribute --> String --> TaskDescription = A normal todo list 
                 d. Add new attribute --> String --> TaskStatus = Pending
                 - Create Item

2. Create Lambda Function:
- Search IAM:
     - Roles --> Create roles --> Service or Use cases: Lambda --> Next --> Select (AmazonAPIGatewayInvokeFullAccess, AmazonDynamoDBFullAccess, AWSLambdaBasicExecutionRole) --> Next
     - Role Name: Serverless_web_service_role --> Create role
- Search Lambda:
     - Create Function --> Function name: TaskManagerFunction
     - Runtime: Python 3.13
     - Execution role: (tick on use an existing role)
     - Existing role: (select) Serverless_web_service_role
     - Create function
     - Code --> lambda_function.py
``` bash
import json
import boto3
from botocore.exceptions import ClientError

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Tasks')

def lambda_handler(event, context):
    print(f"Our event: {event}", "END-Event") 
    operation = event.get('httpMethod')

    if operation == 'GET':
        return get_task(event)
    elif operation == 'POST':
        return create_task(event)
    elif operation == 'PUT':
        return update_task(event)
    elif operation == 'DELETE':
        return delete_task(event)
    else:
        return {
            'statusCode': 400,
            'body': json.dumps('Invalid HTTP Method')
        }

def create_task(event):
    try:
        body = event.get('body')
        if isinstance(body, str):
            body = json.loads(body)
        elif not isinstance(body, dict):
            return {
                'statusCode': 400,
                'body': json.dumps('Invalid request body')
            }
        
        table.put_item(Item=body)
        return {
            'statusCode': 200,
            'body': json.dumps('Task created successfully')
        }
    except (json.JSONDecodeError, ValueError) as e:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': str(e)})
        }
    except ClientError as e:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': e.response['Error']['Message']})
        }

def get_task(event):
    try:
        http_method = event.get('httpMethod')
        query_params = event.get('queryStringParameters', {})
        task_id = query_params.get('TaskID')
        
        if http_method == 'GET' and task_id == 'all':
            print("Getting all Items")
            response = table.scan()
            return {
                'statusCode': 200,
                'body': json.dumps(response['Items'])
            }

        if not task_id:
            return {
                'statusCode': 400,
                'body': json.dumps('TaskID is required')
            }

        response = table.get_item(Key={'TaskID': task_id})
        if 'Item' in response:
            return {
                'statusCode': 200,
                'body': json.dumps(response['Item'])
            }
        else:
            return {
                'statusCode': 404,
                'body': json.dumps('Task not found')
            }
    except ClientError as e:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': e.response['Error']['Message']})
        }

def update_task(event):
    try:
        body = event.get('body')
        if isinstance(body, str):
            body = json.loads(body)
        elif not isinstance(body, dict):
            raise ValueError("Request body is not a valid JSON object")
        
        task_id = body.get('TaskID')
        if not task_id:
            raise ValueError("TaskID is required")

        update_expression = 'SET TaskName = :name, TaskDescription = :desc, TaskStatus = :status'
        expression_values = {
            ':name': body['TaskName'],
            ':desc': body['TaskDescription'],
            ':status': body['TaskStatus']
        }
        
        table.update_item(
            Key={'TaskID': task_id},
            UpdateExpression=update_expression,
            ExpressionAttributeValues=expression_values
        )
        return {
            'statusCode': 200,
            'body': json.dumps('Task updated successfully')
        }
    except (json.JSONDecodeError, ValueError) as e:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': str(e)})
        }
    except ClientError as e:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': e.response['Error']['Message']})
        }

def delete_task(event):
    task_id = event.get('queryStringParameters', {}).get('TaskID')
    if not task_id:
        return {
            'statusCode': 400,
            'body': json.dumps('TaskID is required')
        }

    try:
        table.delete_item(Key={'TaskID': task_id})
        return {
            'statusCode': 200,
            'body': json.dumps('Task deleted successfully')
        }
    except ClientError as e:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': e.response['Error']['Message']})
        }
```
   - Deploy
   - Test --> Event Name: Test
        - Template: apiGateway-aws-proxy
        - In json code mention TaskID = 101 (below query string) --> Save --> Test

3. Search API Gateway:
   - API Name: TaskManagerAPI
   - Create API
   - Create Methods: 
        - Method Type: Get
        - Lambda Function: Attaach to the function we just created
        - URL Query String parameters --> Name: TaskID (tick Required)
   - Create method
   - Integration request --> Edit --> Mapping Template --> Content type: application/json
   - Json code:
``` bash
{
    "httpMethod": "$context.httpMethod",
    "queryStringParameters": {
        #foreach($param in $input.params().querystring.keySet())
            "$param": "$util.escapeJavaScript($input.params().querystring.get($param))"
            #if($foreach.hasNext),#end
        #end
    }
}
```
   - Create Methods: 
           - Method Type: Delete
           - Lambda Function: Attaach to the function we just created
           - URL Query String parameters --> Name: TaskID (tick Required)
      - Create method
      - Integration request --> Edit --> Mapping Template --> Content type: application/json
      - Json code:
   ``` bash
   {
       "httpMethod": "$context.httpMethod",
       "queryStringParameters": {
           #foreach($param in $input.params().querystring.keySet())
               "$param": "$util.escapeJavaScript($input.params().querystring.get($param))"
               #if($foreach.hasNext),#end
           #end
       }
   }
   ```

   - Create Methods: 
           - Method Type: Put
           - Lambda Function: Attaach to the function we just created
           - URL Query String parameters --> Name: TaskID (tick Required)
      - Create method
      - Integration request --> Edit --> Mapping Template --> Content type: application/json
      - Json code:
   ``` bash
   {
    "httpMethod": "$context.httpMethod",
    "body": $input.body
}
   ```
   - Create Methods: 
           - Method Type: POST
           - Lambda Function: Attaach to the function we just created
           - URL Query String parameters --> Name: TaskID (tick Required)
      - Create method
      - Integration request --> Edit --> Mapping Template --> Content type: application/json
      - Json code:
   ``` bash
   {
    "httpMethod": "$context.httpMethod",
    "body": $input.body
}
   ``` 
   - Enable CORS:
        - Gateway response --> Default 5XX & 4XX
        - Access-Control-Allow-Methods --> DELETE, GET, POST, PUT
        - Save
   - Deploy API:
        - *New Stage*
        - Stage Name: prod
        - Deploy
   - Copy Invoke url of prod ( And replace in script.js file )

4. S3 bucket:
   - Create bucket
   - Bucket Name: to-do-bucket-list
   - Untick the Block Public Access
   - Create bucket
   - Goto your created bucket --> Properties --> Static website hosting --> Edit --> Enable
   - Index Document: index.html
   - Save Changes
   - Permissions --> Bucket policy : (Edit)
``` bash
           {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Effect": "Allow",
                  "Principal": "*",
                  "Action": "s3:GetObject",
                  "Resource": "arn:aws:s3:::<your-bucket-name>/*"
              }
          ]
      }
```
   - Save
   - Goto Objects --> Upload files --> Add files (index.html, script.js, style.css) --> Upload
   - Properties --> Bucket Endpoint URL (Copy the url)

5. Final
   - Open New tab in your browser and paste the endpoint url

 Result:
 <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ac9ca4d9-0784-41e9-be0d-e40909f6cfda" />

- You will be able to see this website if the all the configuration is right.
- You can add task and delete the task as per TaskID.
  

   

           
