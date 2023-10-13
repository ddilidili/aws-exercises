# PokeDex on AWS Lambda
![PokeDex](https://static.wikia.nocookie.net/pokemon/images/5/5c/Gen_I_Pokedex.png/revision/latest?cb=20100717083120)

## Requirements
Access to a sandbox environment. AWS re/Start students may use `1-[CF]-Lab - Sandbox Environment` for this activity.

## Task 1: Create a Lambda Function
1. Go to Lambda dashboard by typing `Lambda` on the search bar.
1. On `Functions` page, click `Create function`.
1. Select `Author from scratch` and input the following values:
   - Function Name: PokeDex
   - Runtime: Python 3.10
   - Architecture: x84_64
   - Execution Role: Use an existing role (LabsRole)
1. Under `Advanced Settings`, check `Enable function URL`.

## Task 2: Add your Python codes
1. Rename `lambda_function.py` to `main.py`.
1. Scroll down a bit and look for `Runtime Settings`. Change `Handler` from `lambda_function.lambda_handler` to `main.app`.
1. Inside `main.py`, paste the code below:

   ```python
   import json
   import urllib3

   def app(event, context):
       http = urllib3.PoolManager()
       response = http.request(
           "GET", 
           "https://pokeapi.co/api/v2/pokemon{}".format(
               event['requestContext']['http']['path']
           )
       )
       pokemon = json.loads(response.data)
       body = {
           "name": pokemon["name"],
           "number": pokemon["order"],
       }
       return {
           'statusCode': 200,
           'body': body   
       }
   ```
2. Click `Deploy`.

## Task 3: Check your output
1.  Under `Configuration` tab, look for `Function URL` and open the link to a new tab/window and add a Pokemon name after a slash.