# Technical Guide

## Overview

> InvoiceBldr is created as a cluster of 4 microservices in a loosely coupled architecture. Each microservice is deployed as a containerized unit in an [AWS Elastic Beanstalk (EBS)](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html) application environment. EBS provides a level of abstraction on top of AWS's Compute servce, [EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html). EBS offers managed platforms for deploying and maintaining web servers, streamlining the instance configuration and maintainance process.

> The server-side microservices share and maintain a state and progressively update the single-page front-end application using push notifications.


> The server-side nodes utilize AWS SQS to establish messaging queues, send, recieve, and finally process requests by querying the database. For more information on using SQS, visit the [SQS Messaging](/tech/#messaging-with-aws-sqs) section of this guide.

> <u>Server-side Microservices:</u>
> <ul>
>   <li>Invoice Processing</li>
>   <li>Notifications</li>
>   <li>Logs and Reports</li>
> </ul>

## Frameworks

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

> <strong>Note: </strong>Flask applications <strong>must</strong> run within a virtual environment in order to manage dependencies. As a result, certain software and utilities, such as the Google API client, must be installed within the virtual environment. This example demonstrates the steps needed to install the [Google API Client Library](https://developers.google.com/api-client-library):

    # Load the environment's /activate script into the shell
    source ~/.virtualenvs/name-of-environment/activate
    # install the module using pip3
    python3-pip install -I google-api-python-client

> A <em>pipenv</em> virtual environment can be removed by using the `-rm` command:

    pipenv --rm

> If the virtual environment is removed, `pipenv install` and `pipenv shell` commands are required before the Flask application can be restarted.

## Interacting with AWS S3

> All original invoice documents and supporting attachments are uploaded to an [S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html) bucket using the [AWS Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) client and stored in a [Glacier Deep Archive](https://aws.amazon.com/glacier/).

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

print(response)

```
> The example above creates a subdirectory for the unique `admin_id`, and another nested subdirectory for each `client_id`. The upload object can subsequently be retrieved by passing the entire key as its absolute path. Subdirectories and objects are created only if they do not currently exist within the bucket. If an object is updated, a new version of the object is saved in the S3 bucket

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
response = s3_client.get_object(Bucket='invoice-docs', Key=f'{admin_id}/{client_id}/{invoice_number}.csv')

print(response)
```

## Messaging with AWS SQS

### Overview

> [AWS SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html) provides secure messaging queues that allow integration between decoupled or loosely coupled software systems.
> SQS plays an essential role in implementing InvoiceBldr as a collection of microservices in a decoupled design architecture. 

> While SQS allows for both standard and FIFO queue configurations, InvoiceBldr uses a standard configuration to communicate between services. Doing so allows the application microservices to utilize the <strong>At Least Once Delivery</strong> feature.

> <strong>At Least Once Delivery</strong> ensures communication reliability between decoupled services. If a message fails or is rejected by the reciever, it can be re-captured by the SQS queue, processed, and then re-sent.

### Example

> This example demonstrates using the `sendMessage` method from the SQS Javascript SDK to create and save an invoice note.

    /* the message body will include the admin userId and the client userId, 
    the invoice number, and the text of the note */
    const body = {
        adminId: '12345',
        clientId:  '67890',
        invoiceNumber: '13579'
        body: 'This is a sample note.'
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


> The following example demonstrates using the `recieve_message` method of the SQS Javascript SDK to recieve a message form a target queue.

```
const sqs = new AWS.SQS (
    {
        endpoint: 'sqs.us-west-2.amazonaws.com'
        accessKeyId: 'AWS_ACCESS_KEY',
        secretAccessKey: 'AWS_SECRET_KEY'
    }
);

response = sqs.receive_message(
    QueueUrl='/123456789012/InvoiceNotes',
    AttributeNames=['All'],
)

console.log(response)
```
