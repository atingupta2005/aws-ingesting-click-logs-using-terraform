# Ingesting Web Application Click Logs into AWS using Terraform (By HashiCorp)

- API based ingestion application system for websites & applications to push user interactions, click actions from their website into AWS

The following steps provide an overview of this implementation:

1. Java source build – Provided code is packaged & build using Apache Maven (https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

2. Terraform commands are initiated (provided below) to deploy the infrastructure in AWS. 
3. An API Gateway, S3 bucket, Dynamo table, following Lambdas are built and deployed in AWS.
   - Lambda Authorizer – This lambda validates the incoming request for header authorization from API gateway to processing lambda. 
        - ClickLogger Lamba – This lambda processes the incoming request and pushes the data into Firehose stream
        - Transformational Lambda – This lambda listens to the Firehose stream data and processes this to DynamoDB. In real world these lambda can more additional filtering, processing etc.,
   - Once the data “POST” is performed to the API Gateway exposed endpoint, the data traverses through the lambda and Firehose stream converts the incoming stream into a Parquet file. We use AWS Glue to perform this operation.
   - The incoming click logs are eventually saved as Parquet files in S3 bucket and additionally in the DynamoDB

![Alt text](ingesting%20click%20logs%20from%20web%20application.png?raw=true "Title")

### Prerequisites

    - Make sure to have Java installed and running on your machine. For instructions, see Java Development Kit (https://www.oracle.com/java/technologies/javase-downloads.html)
    - Set up Terraform. For steps, see Terraform downloads (https://www.terraform.io/downloads.html).

### Steps

1. Clone this repository and execute the below command to spin up the infrastructure and the application

2. Execute the below commands

    ```
    git clone https://github.com/atingupta2005/aws-ingesting-click-logs-using-terraform
    cd aws-ingesting-click-logs-using-terraform
    cd source/clicklogger
    mvn clean package
    cd ../../terraform/templates    
    terraform init
    terraform plan
    terraform apply -auto-approve
    ```

### Test

1. In AWS Console, select “API Gateway”. Select “click-logger-api” 
2. Select “Stages” on the left pane
3. Click “dev” > “POST” (within the “/clicklogger” route)
4. Copy the invoke Url. A sample url will be like this -  https://qvu8vlu0u4.execute-api.us-east-1.amazonaws.com/dev/clicklogger
5. Use REST API tool like Postman or Chrome based web extension like RestMan to post data to your endpoint
    - Request Type: POST
    - Body Type: RAW JSON
    - Add Header: Key “Authorization” with value “ALLOW=ORDERAPP”.

Sample Json Request:
```
{
  "requestid": "OAP-guid-05122020-1345-12345-678910",
  "contextid": "OAP-guid-05122020-1345-1234-5678",
  "callerid": "OrderingApplication",
  "component": "login",
  "action": "click",
  "type": "webpage"
}
```
6. Output - You should see the output in both S3 bucket and DynamoDB
    - S3 – Navigate to the bucket created as part of the stack
        * Select the file and view the file from “Select From” sub tab . You should see something ingested stream got converted into parquet file.
        * Select the file and view the data
    - DynamoDB table - Select “clickloggertable” and view the “items” to see data. 
 
 ## Cleanup
    ```
    # CLI Commands to delete the S3  
    terraform state show aws_s3_bucket.click_logger_firehose_delivery_s3_bucket | grep clicklogger-dev-firehose-delivery-bucket-
    aws s3 rb s3://clicklogger-dev-firehose-delivery-bucket-<your-account-number> --force
    terraform destroy –-auto-approve
    ```


## License

This library is licensed under the MIT-0 License. See the LICENSE file.
