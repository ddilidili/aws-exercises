# PokeDex on AWS Lambda
![PokeDex](https://static.wikia.nocookie.net/pokemon/images/5/5c/Gen_I_Pokedex.png/revision/latest?cb=20100717083120)

## Requirements
Access to a sandbox environment. AWS re/Start students may use `1-[CF]-Lab - Sandbox Environment` for this activity.

## Architecture
![Lambda](../../assets/003-s3.png)

## Task 1: Create an S3 bucket
1. Go to S3 dashboard by typing `S3` on the search bar.
1. Create a bucket with a unique name. Suggestion: `pokedex-<name>-phman17`.
1. Click `Create bucket`.
1. Upload `sample-data.txt` to the bucket.

## Task 2: Create a Lambda function
1. Go to Lambda dashboard by typing `Lambda` on the search bar.
1. On `Functions` page, click `Create function`.
1. Select `Author from scratch` and input the following values:
   - Function Name: pokedex-write
   - Runtime: Python 3.10
   - Architecture: x84_64
   - Execution Role: Use an existing role (LabsRole)
1. On the `pokedex-write` page, click `Add trigger`
1. Select `S3` from the list and type the name of the bucket you created earlier.
1. Other details:
   - Event types: PUT
   - Suffix: .txt
1. Click `Add`


## Task 4: Add your Python codes
1. Rename `lambda_function.py` to `main.py`.
1. Scroll down a bit and look for `Runtime Settings`. Change `Handler` from `lambda_function.lambda_handler` to `main.app`.
1. Inside `main.py`, paste the code below:
   ```python
   import json
   import boto3
   import urllib3

   def app(event, context):
       bucket = event['Records'][0]['s3']['bucket']['name']
       key = event['Records'][0]['s3']['object']['key']
       names = get_list(bucket, key)

   def get_list(bucket, key):
       s3 = boto3.client('s3')
       response = s3.get_object(Bucket=bucket, Key=key)
       data = response['Body'].read().decode().split("\n")
       for pokemon in data:
           info = get_info(pokemon)
           write_table(info)
           
   def get_info(pokemon):
       http = urllib3.PoolManager()
       response = http.request(
           "GET", 
           "https://pokeapi.co/api/v2/pokemon/{}".format(
              pokemon
           )
       )
       pokemon = json.loads(response.data)
       return {
           "name": pokemon['name'],
           "order": pokemon['order'],
           "types": [p_type['type']['name'] for p_type in pokemon['types']]
       }
       
   def write_table(info):
       # https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/index.html
       # https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/table/put_item.html
       dynamodb = boto3.resource('dynamodb')
       table = dynamodb.Table('pokedex')
       response = table.put_item(Item=info)
       if response['ResponseMetadata']['HTTPStatusCode'] != 200:
           raise Exception('Unable to add item to table!')
   ```
1. Click `Deploy`.
1. Test the function by creating a test event called `S3 Put`.
1. Change the following details:
   - Bucket Name: Name of the bucket you created earlier
   - Object Key: sample-data.txt
1. Click `Test` and check your DDB table.