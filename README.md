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
     
                 b.  Add new attribute --> String --> TaskName = ITR Filling
                 - Add new attribute --> String --> TaskDescription = A normal todo list 
                 - Add new attribute --> String --> TaskStatus = Pending
                 - Create Item
     
