# UBA -  Code for the connection of android application with the cloud.

Dyanmo Code :


import json
import boto3
import random

dynamodb = boto3.resource('dynamodb')
table_name = 'tester'
accounts_table_name = 'Accounts'
table = dynamodb.Table(table_name)
accounts_table = dynamodb.Table(accounts_table_name)

def lambda_handler(event, context):
    StatusCode = int(event['queryStringParameters']['StatusCode'])

    if StatusCode == 100:
        uid = event['queryStringParameters']['uid']
        response = table.get_item(Key={'uid': uid})
        item = response.get('Item', {})

        name = item.get('name')
        password = item.get('password')
        isvalid = item.get('isvalid')

        transactionResponse = {
            'StatusCode': StatusCode,
            'uid': uid,
            'name': name,
            'password': password,
            'isvalid': isvalid
        }

    elif StatusCode == 101:
        uid = event['queryStringParameters']['uid']
        name = event['queryStringParameters']['name']
        password = event['queryStringParameters']['password']
        isvalid = 'yes'

        item = {
            'uid': uid,
            'isvalid': isvalid,
            'name': name,
            'password': password
        }

        try:
            table.put_item(Item=item)
            request = 'success'
        except:
            request = 'failure'

        transactionResponse = {
            'StatusCode': StatusCode,
            'request': request,
            'uid': uid,
            'password': password
        }

    elif StatusCode == 111:
        uid = event['queryStringParameters']['uid']
        source_acc = event['queryStringParameters']['source_acc']
        destination_acc = event['queryStringParameters']['destination_acc']
        amount = int(event['queryStringParameters']['amount'])
        atmno = event['queryStringParameters']['atmno']

        response = accounts_table.get_item(Key={'Acc_no': source_acc})
        source_account = response.get('Item', {})

        fetched_atmno = source_account.get('atmno')
        fetched_balance = source_account.get('balance', 0)

        response = accounts_table.get_item(Key={'Acc_no': destination_acc})
        destination_account = response.get('Item', {})

        if atmno == fetched_atmno:
            if amount <= fetched_balance:
                fetched_balance -= amount
                fetched_balance_dest = destination_account.get('balance', 0) + amount

                accounts_table.update_item(
                    Key={'Acc_no': source_acc},
                    UpdateExpression='SET balance = :value',
                    ExpressionAttributeValues={
                        ':value': fetched_balance
                    }
                )

                accounts_table.update_item(
                    Key={'Acc_no': destination_acc},
                    UpdateExpression='SET balance = :value',
                    ExpressionAttributeValues={
                        ':value': fetched_balance_dest
                    }
                )

                transactionResponse = {
                    'StatusCode': StatusCode,
                    'request': 'successful',
                    'uid': uid,
                    'trans_status': 'success'
                }
            else:
                transactionResponse = {
                    'StatusCode': StatusCode,
                    'request': 'failed',
                    'uid': uid,
                    'trans_status': 'insufficient_balance'
                }
        else:
            transactionResponse = {
                'StatusCode': StatusCode,
                'request': "no",
                'uid': uid,
                'trans_status': 'invalid_atmno'
            }

    elif StatusCode == 136:
        uid = event['queryStringParameters']['uid']
        bank = event['queryStringParameters']['bank']
        branch = event['queryStringParameters']['branch']
        accno = event['queryStringParameters']['accno']
        ifsc = event['queryStringParameters']['ifsc']
        phno = event['queryStringParameters']['phno']
        atmno = event['queryStringParameters']['atmno']
        random_number = random.randint(10000, 50000)

        item = {
            'uid': uid,
            'bank': bank,
            'branch': branch,
            'Acc_no': accno,
            'ifsc': ifsc,
            'phno': phno,
            'atmno': atmno,
            'balance': random_number
        }

        try:
            accounts_table.put_item(Item=item)
            request = 'success'
        except:
            request = 'failure'

        transactionResponse = {
            'StatusCode': StatusCode,
            'request': request,
            'uid': uid
        }
        
    elif(StatusCode == 102):

        '''
        StatusCode = 102

        queryStringParameters = {
        
        status code = 102
        uid = 'abc@abc.com'
        accno ="asdb234"
        atmno = 8456
        
        }
    
        responce params = {
        status code = 102
        request status = 'successfull'
        uid = 'abc@abc.com'
        
        invoke url : https://e5hbne06n1.execute-api.ap-south-1.amazonaws.com/test/crud?StatusCode=102&accno=526698

        }
    
        '''
        # step 1 : parse the query string params

        
        accno = event['queryStringParameters']['accno']

        # step 2 : Contrruct the body of the response object

        transactionResponse = {}  #this is an empty object in which we fill the values for the json response object
        
        #fill the values

        response = accounts_table.get_item(Key={'Acc_no': accno})
        item = response.get('Item', {})

        fetched_atmno = item.get('atmno')
        fetched_uid = item.get('uid')
        

        

        transactionResponse['StatusCode'] = StatusCode
        transactionResponse['uid'] = fetched_uid
        transactionResponse['atmno'] = fetched_atmno
        transactionResponse['accno'] = accno
        
    elif(StatusCode == 103):

        '''
        StatusCode = 103

        queryStringParameters = {
        
        status code = 103
        accno ="asdb234"
        
        }
    
        responce params = {
        status code = 103
         transactionResponse['StatusCode'] = StatusCode
        transactionResponse['uid'] = fetched_uid
        transactionResponse['atmno'] = fetched_atmno
        transactionResponse['accno'] = accno
        transactionResponse['bank'] = fetched_bank
        transactionResponse['branch'] = fetched_branch
        transactionResponse['ifsc'] = fetched_ifsc
        transactionResponse['phno'] = fetched_phno
        
        

        invoke url : https://e5hbne06n1.execute-api.ap-south-1.amazonaws.com/test/crud?StatusCode=102&accno=526698

        }
    
        '''
        # step 1 : parse the query string params

        accno = event['queryStringParameters']['accno']

        # step 2 : Contrruct the body of the response object

        transactionResponse = {}  #this is an empty object in which we fill the values for the json response object
        
        #fill the values

        response = accounts_table.get_item(Key={'Acc_no': accno})
        item = response.get('Item', {})

        fetched_atmno = item.get('atmno')
        fetched_uid = item.get('uid')
        fetched_bank = item.get('bank')
        fetched_branch = item.get('branch')
        fetched_ifsc = item.get('ifsc')
        fetched_phno = item.get('phno')

        

        transactionResponse['StatusCode'] = StatusCode
        transactionResponse['uid'] = fetched_uid
        transactionResponse['atmno'] = fetched_atmno
        transactionResponse['accno'] = accno
        transactionResponse['bank'] = fetched_bank
        transactionResponse['branch'] = fetched_branch
        transactionResponse['ifsc'] = fetched_ifsc
        transactionResponse['phno'] = fetched_phno
        
    elif(StatusCode == 118):

        '''
        StatusCode = 118

        queryStringParameters = {
        
        status code = 118
        accno ="asdb234"
        p
        
        }
    
        responce params = {
        status code = 118
        request status = 'successfull'
        uid = 'abc@abc.com'

        invoke url : https://e5hbne06n1.execute-api.ap-south-1.amazonaws.com/test/crud?StatusCode=102&accno=526698

        }
    
        '''
        # step 1 : parse the query string params

        accno = event['queryStringParameters']['accno']
        atmno = event['queryStringParameters']['atmno']

        # step 2 : Contrruct the body of the response object

        transactionResponse = {}  #this is an empty object in which we fill the values for the json response object
        
        #fill the values

        response = accounts_table.get_item(Key={'Acc_no': accno})
        item = response.get('Item', {})

        fetched_atmno = item.get('atmno')
        fetched_uid = item.get('uid')
        
        request = "Deletion Failed"
        
        
        if(atmno == fetched_atmno):
            
            
            accounts_table.delete_item(Key={'Acc_no': accno})
            request = 'success'
            
            
            
            
            
            request = "successful"
        

    
        
        transactionResponse['StatusCode'] = StatusCode
        transactionResponse['uid'] = fetched_uid
        transactionResponse['atmno'] = fetched_atmno
        transactionResponse['accno'] = request
        
    
    elif(StatusCode == 160):

        '''
        StatusCode = 102

        queryStringParameters = {
        
        status code = 102
        uid = 'abc@abc.com'
        accno ="asdb234"
        atmno = 8456
        
        }
    
        responce params = {
        status code = 102
        request status = 'successfull'
        uid = 'abc@abc.com'
        
        invoke url : https://e5hbne06n1.execute-api.ap-south-1.amazonaws.com/test/crud?StatusCode=102&accno=526698

        }
    
        '''
        
        
        # step 1: parse the query string params
        accno = event['queryStringParameters']['accno']

        # step 2: Construct the body of the response object
        transactionResponse = {}  # empty object to store the response

        # Fetch the account balance from the DynamoDB table
        response = accounts_table.get_item(Key={'Acc_no': accno})
        item = response.get('Item', {})
        fetched_balance = str(item.get('balance'))
        fetched_atmno = item.get('atmno')  # Fetch the ATM number

        # Fill the values in the response object
        transactionResponse['StatusCode'] = StatusCode
        transactionResponse['uid'] = fetched_balance  # Store the balance instead of UID
        transactionResponse['atmno'] = fetched_atmno
        transactionResponse['accno'] = accno
        
    elif(StatusCode == 165):

        '''
        StatusCode = 102

        queryStringParameters = {
        
        status code = 102
        uid = 'abc@abc.com'
        accno ="asdb234"
        atmno = 8456
        
        }
    
        responce params = {
        status code = 102
        request status = 'successfull'
        uid = 'abc@abc.com'
        
        invoke url : https://e5hbne06n1.execute-api.ap-south-1.amazonaws.com/test/crud?StatusCode=102&accno=526698

        }
    
        '''
        
        
        # step 1: parse the query string params
        accno = event['queryStringParameters']['accno']

        # step 2: Construct the body of the response object
        transactionResponse = {}  # empty object to store the response

        # Fetch the account balance from the DynamoDB table
        response = accounts_table.get_item(Key={'Acc_no': accno})
        item = response.get('Item', {})
        fetched_bank = item.get('bank')
        fetched_branch = item.get('branch')
        fetched_ifsc = item.get('ifsc')
        fetched_phone = item.get('phno')
        fetched_atmno = item.get('atmno')  # Fetch the ATM number

        # Fill the values in the response object
        transactionResponse['StatusCode'] = StatusCode
        transactionResponse['bank'] = fetched_bank  
        transactionResponse['branch'] = fetched_branch  
        transactionResponse['ifsc'] = fetched_ifsc 
        transactionResponse['phone'] = fetched_phone 
        transactionResponse['atmno'] = fetched_atmno
        transactionResponse['accno'] = accno

    responseObject = {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps(transactionResponse)
    }

    return responseObject
