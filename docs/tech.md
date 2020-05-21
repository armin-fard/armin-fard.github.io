# Technical Guide

## Overview

> InvoiceBldr is built as a cluster of 4 microservices in a loosely coupled architecture. Each microservice is deployed as a containerized unit on an AWS Elastic Beanstalk (EBS) instance.

> The server-side microservices share and maintain a state and progressively update the single-page, client UI via push notifications.


> Microservices utilize AWS SQS to establish messaging queues, send, recieve, and finally process requests by querying the database. For more information on using SQS, visit the [SQS Messaging](/tech/#messaging-with-aws-sqs) section of this guide.

> <u>Server-side Microservices:</u>
> <ul>
>   <li>Invoice processing</li>
>   <li>Websocket push notifications</li>
>   <li>Logs and reports</li>
> </ul>

## Framework

> <u>UI Microservice</u>

> <ul>
>    <li><a href="https://www.javascript.com/">JavaScript</a></li>
>    <li><a href="https://vuejs.org/">Vue JS</a></li>
>    <li><a href="https://vuex.vuejs.org/">Vuex Store</a></li>
>    <li><a href="https://vuetifyjs.com/">Vuetify</a></li>
> </ul>


> <em>*The client side application is deployed on AWS S3, using S3's static web hosting feature.</em>

> <u>Server-side Microservices</u>

> <ul>
>   <li><a href="https://www.python.org/">Python 3.7</a></li>
>    <li><a href="https://palletsprojects.com/p/flask/">Flask</a></li>
> </ul>
> </br>


## Getting Started

```
    # Server set-up and installation
    pipenv install      # Install dependencies from Pipfile and create a virtual environment
    pipenv shell        # Access the environment shell
    python app.py       # Start the development server

    # Client set-up and installation
    npm install         # Install dependencies from package.json
    npm start           # Start the development server
```

> <strong>Note: </strong>Flask applications run inside a virtual environment in order to manage dependencies. As a result, certain software and utilities, such as the Google API client, must be installed within the virtual environment. This example demonstrates the steps needed to install the Google API client library:

    # Load the environment's /activate script into the shell
    source ~/.virtualenvs/name-of-environment/activate
    # install the module using pip3
    python3-pip install -I google-api-python-client

> Adding `-I` will ensure that the module is installed within the virtual environment.

## Interacting with AWS S3

> All original invoice documents and supporting attachments are uploaded via the AWS boto3 client and stored in a [Glacier Deep Archive](https://aws.amazon.com/glacier/).

> S3 does not recognize a directory hierarchy, and instead identifies each object with a unique key that signifies its absolute path.
> This example demonstrates how object keys are used to store documents in different sub directories.

### Uploading an Object

```
import boto3
from dotenv import load_dotenv

# the accessKey and secrecKey are defined in the /.env file
accessKey = os.getenv('accessKey')
secretKey = os.getenv('secretKey')

# create the boto3 s3 client service
s3_client = boto3.client('s3', aws_access_key_id=accessKey, aws_secret_access_key=secretKey)

# the put_object method adds an object to an S3 bucket based on the object-key passed 
response = s3_client.put_object(
    ACL= 'public-read', # allows public access to the object
    Bucket='invoice-docs', # the bucket for storing invoice-related documents
    Body=body, # the binary file object parsed from the request body
    Key=f'csv-invoices/{admin_id}/{client_id}/{invoice_number}.csv' # identifies the absolute path of the object, including any subdirectories
)

```
> The example above creates a subdirectory for the unique `admin_id`, and another nested subdirectory for each `client_id`. The object can be retrieved by using the entire key as its absolute path. Subdirectories and objects are created only if they do not currently exist within the bucket

> <strong>Note: </strong>The keys for invoice file objects and subdirectories must be created using this naming convention in order to allow for unifrom accessibility across the application.

### Downloading an Object

> the `get_object` method accepts `(Bucket='name-of-bucket', Key='object-key')` as arguments.
> Here,`Key` refers the object's full key, including any nested subdirectories.

```
import boto3

accessKey = os.getenv('accessKey')
secretKey = os.getenv('secretKey')

s3_client = boto3.client('s3', aws_access_key_id=accessKey, aws_secret_access_key=secretKey)

# Call the get_object method and save the response in a variable
invoice = s3_client.get_object(Bucket='invoice-docs', Key=f'{admin_id}/{client_id}/{invoice_number}.csv')

```

## Messaging with AWS SQS

### Overview

> [AWS SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html) provides secure messaging queues that allow integration between decoupled or loosely coupled software systems.
> SQS plays an essential role in implementing InvoiceBldr as a collection microservices in a decoupled design architecture. 

> While SQS allows for both standard and FIFO queue configurations, InvoiceBldr uses the standard configuration to communicate between services. Using the standard queue allows for <strong>At Least Once Delivery</strong>.

> <strong>At Least Once Delivery</strong> ensures that every message is delivered at least once. However, some messages may be delivered more than once. 
> This allows for more reliability in transmitting messages and requests between services, ensuring that an error occuring on the reciever does not result in a lost message.

### Example

> This example demonstrates using the `sendMessage` method of the SQS Javascript SDK to create an invoice note.

    /* the message body will include the admin's user ID, the client's user ID, 
    the invoice number, and the text of the note */
    const body = {
        adminId: '12345',
        clientId:  '67890',
        invoiceNumber: '13579'
        note: 'This is a sample note.'
    }

    /* create an SQS client using the AWS SDK */
    const sqs = new AWS.SQS (
        {
            endpoint: 'sqs.us-west-2.amazonaws.com'
            accessKeyId: 'AWS_ACCESS_KEY',
            secretAccessKey: 'AWS_SECRET_KEY'
        }
    );

    const params = {
        MessageBody: body, /* SQS accepts JSON, XML and plain text as MessageBody */
        QueueUrl: '/123456789012/InvoiceNotes',
    };

    sqs.sendMessage(params, (err, data) => {
        if (err) {
            console.log(err, err.stack)
        } else {
            console.log(data)
        }
    });
