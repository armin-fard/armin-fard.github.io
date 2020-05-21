# VPC Model

><em>The following diagram is a visualization of the InvoiceBldr AWS VPC.</em>

![VPC](/img/InvoiceBldr.png "VPC Diagram")

> <ul>
> <li>Each microservice EBS instance is deployed using two subnets in two availability zones in order to configure Application Load Balancers.</li>

> <li>The Vue app communicates with each microservice through the managed SQS queues.</li>

> <li>The microservice dedicated to invoice processing sends but does not recieve messages from the other server-side microservices.</li> 
> </ul>
