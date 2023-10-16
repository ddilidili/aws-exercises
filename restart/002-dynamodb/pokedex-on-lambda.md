# PokeDex on AWS Lambda
![PokeDex](https://static.wikia.nocookie.net/pokemon/images/5/5c/Gen_I_Pokedex.png/revision/latest?cb=20100717083120)

## Requirements
Access to a sandbox environment. AWS re/Start students may use `1-[CF]-Lab - Sandbox Environment` for this activity.

## Architecture
![Lambda](../../assets/002-dynamodb.png)

## Task 1: Create a DynamoDB table
1. Go to DynamoDB dashboard by typing `DynamoDB` on the search bar.
1. On `Tables` page, click `Create table`.
1. Enter the following under `Table details`:
    - Table Name: pokedex
    - Partition Key: name (string)
    - Sort Key: order (number)
1. Under `Table Settings`, choose `Customized Settings`
1. Under `Table Class`, choose `DynamoDB Standard`
1. Under `Read/write capacity settings`, choose `On-demand`
1. Click `Create table`.

## Task 2: Add an entry to the table
1. From the list of tables in `Tables` page, click the link to your table.
1. Under `Actions`, select `Create item`.
1. For `name` amd `order`, type `bulbasaur` and `1` respectively.
1. `Add new attribute` by selecting `list` from the dropdown.
1. Name the attribute as `types`
1. Add two string fields: `grass` and `poison`.
1. Click `Create item`

## Task 3: Create a Lambda Function
1. Go to Lambda dashboard by typing `Lambda` on the search bar.
1. On `Functions` page, click `Create function`.
1. Select `Author from scratch` and input the following values:
   - Function Name: pokedex-read
   - Runtime: Python 3.10
   - Architecture: x84_64
   - Execution Role: Use an existing role (LabsRole)
1. Under `Advanced Settings`, check `Enable function URL`. Set Auth type to `NONE`.

## Task 4: Add your Python codes
1. Rename `lambda_function.py` to `main.py`.
1. Scroll down a bit and look for `Runtime Settings`. Change `Handler` from `lambda_function.lambda_handler` to `main.app`.
1. Inside `main.py`, paste the code below:
   ```python
   import json
   import boto3

   def app(event, context):
       body = get_items()
       return {
           'statusCode': 200,
           'body': body
       }

   def get_items():
       # https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/index.html
       dynamodb = boto3.resource('dynamodb')
       table = dynamodb.Table('pokedex')
       response = table.scan(
           Select='ALL_ATTRIBUTES',
       )
       return response['Items']
   ```
1. Click `Deploy`.
1. Check output of your function using your `Function URL`.