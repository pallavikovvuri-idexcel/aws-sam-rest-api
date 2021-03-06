# AWS SAM REST API

this is sample CRUD api for AWS Lambda + DynamoDB.

## validate template

```
$ sam validate --template template.yaml
2019-07-20 16:41:47 Found credentials in shared credentials file: ~/.aws/credentials
```

## setup

start DynamoDB local

```
$ docker-compose up
```

create table

```
$ export DYNAMODB_LOCAL_ENDPOINT=http://192.168.1.3:8002
$ node api/create_table.js
```

## start-api

```
$ sam local start-api
2019-07-21 22:43:03 Mounting MovieApiFunction at http://127.0.0.1:3000/movies/{id}/delete [DELETE]
2019-07-21 22:43:03 Mounting MovieApiFunction at http://127.0.0.1:3000/movies/create [POST]
2019-07-21 22:43:03 Mounting MovieApiFunction at http://127.0.0.1:3000/movies [GET]
2019-07-21 22:43:03 You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions, changes will be reflected instantly/automatically. You only need to restart SAM CLI if you update your AWS SAM template
2019-07-21 22:43:03  * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit)
```

```
$ curl -XPOST http://127.0.0.1:3000/movies/create -d '{"year":2019, "title":"アラジン"}'
{"message":"success","result":{}}
```

```
$ curl -XGET http://127.0.0.1:3000/movies
{"message":"success","result":{"Items":[{"title":"title","year":2019},{"title":"アベンジャーズ/エンドゲーム","year":2019,"url":"https://marvel.disney.co.jp/movie/avengers-endgame.html"},{"title":"アラジン","year":2019}],"Count":3,"ScannedCount":3}}
```

## ref

[Node.js と DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/GettingStarted.NodeJs.01.html)

---

# aws-sam-rest-api

This is a sample template for aws-sam-rest-api - Below is a brief explanation of what we have generated for you:

```bash
.
├── README.MD                   <-- This instructions file
├── event.json                  <-- API Gateway Proxy Integration event payload
├── hello-world                 <-- Source code for a lambda function
│   └── app.js                  <-- Lambda function code
│   └── package.json            <-- NodeJS dependencies and scripts
│   └── tests                   <-- Unit tests
│       └── unit
│           └── test-handler.js
├── template.yaml               <-- SAM template
```

## Requirements

- AWS CLI already configured with Administrator permission
- [NodeJS 10.10+ installed](https://nodejs.org/en/download/releases/)

- [Docker installed](https://www.docker.com/community-edition)

## Setup process

### Local development

**Invoking function locally using a local sample payload**

```bash
sam local invoke HelloWorldFunction --event event.json
```

**Invoking function locally through local API Gateway**

```bash
sam local start-api
```

If the previous command ran successfully you should now be able to hit the following local endpoint to invoke your function `http://localhost:3000/hello`

**SAM CLI** is used to emulate both Lambda and API Gateway locally and uses our `template.yaml` to understand how to bootstrap this environment (runtime, where the source code is, etc.) - The following excerpt is what the CLI will read in order to initialize an API and its routes:

```yaml
---
Events:
  HelloWorld:
    Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
    Properties:
      Path: /hello
      Method: get
```

## Packaging and deployment

AWS Lambda NodeJS runtime requires a flat folder with all dependencies including the application. SAM will use `CodeUri` property to know where to look up for both application and dependencies:

```yaml
...
    HelloWorldFunction:
        Type: AWS::Serverless::Function
        Properties:
            CodeUri: hello-world/
            ...
```

Firstly, we need a `S3 bucket` where we can upload our Lambda functions packaged as ZIP before we deploy anything - If you don't have a S3 bucket to store code artifacts then this is a good time to create one:

```bash
aws s3 mb s3://BUCKET_NAME
```

Next, run the following command to package our Lambda function to S3:

```bash
sam package \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME
```

Next, the following command will create a Cloudformation Stack and deploy your SAM resources.

```bash
sam deploy \
    --template-file packaged.yaml \
    --stack-name aws-sam-rest-api \
    --capabilities CAPABILITY_IAM
```

> **See [Serverless Application Model (SAM) HOWTO Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-quick-start.html) for more details in how to get started.**

After deployment is complete you can run the following command to retrieve the API Gateway Endpoint URL:

```bash
aws cloudformation describe-stacks \
    --stack-name aws-sam-rest-api \
    --query 'Stacks[].Outputs[?OutputKey==`HelloWorldApi`]' \
    --output table
```

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called sam logs. sam logs lets you fetch logs generated by your Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
sam logs -n HelloWorldFunction --stack-name aws-sam-rest-api --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Testing

We use `mocha` for testing our code and it is already added in `package.json` under `scripts`, so that we can simply run the following command to run our tests:

```bash
cd hello-world
npm install
npm run test
```

## Cleanup

In order to delete our Serverless Application recently deployed you can use the following AWS CLI Command:

```bash
aws cloudformation delete-stack --stack-name aws-sam-rest-api
```

## Bringing to the next level

Here are a few things you can try to get more acquainted with building serverless applications using SAM:

### Learn how SAM Build can help you with dependencies

- Uncomment lines on `app.js`
- Build the project with `sam build --use-container`
- Invoke with `sam local invoke HelloWorldFunction --event event.json`
- Update tests

### Create an additional API resource

- Create a catch all resource (e.g. /hello/{proxy+}) and return the name requested through this new path
- Update tests

### Step-through debugging

- **[Enable step-through debugging docs for supported runtimes](<(https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-debugging.html)>)**

Next, you can use AWS Serverless Application Repository to deploy ready to use Apps that go beyond hello world samples and learn how authors developed their applications: [AWS Serverless Application Repository main page](https://aws.amazon.com/serverless/serverlessrepo/)

# Appendix

## Building the project

[AWS Lambda requires a flat folder](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-create-deployment-pkg.html) with the application as well as its dependencies in a node_modules folder. When you make changes to your source code or dependency manifest,
run the following command to build your project local testing and deployment:

```bash
sam build
```

If your dependencies contain native modules that need to be compiled specifically for the operating system running on AWS Lambda, use this command to build inside a Lambda-like Docker container instead:

```bash
sam build --use-container
```

By default, this command writes built artifacts to `.aws-sam/build` folder.

## SAM and AWS CLI commands

All commands used throughout this document

```bash
# Invoke function locally with event.json as an input
sam local invoke HelloWorldFunction --event event.json

# Run API Gateway locally
sam local start-api

# Create S3 bucket
aws s3 mb s3://BUCKET_NAME

# Package Lambda function defined locally and upload to S3 as an artifact
sam package \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME

# Deploy SAM template as a CloudFormation stack
sam deploy \
    --template-file packaged.yaml \
    --stack-name aws-sam-rest-api \
    --capabilities CAPABILITY_IAM

# Describe Output section of CloudFormation stack previously created
aws cloudformation describe-stacks \
    --stack-name aws-sam-rest-api \
    --query 'Stacks[].Outputs[?OutputKey==`HelloWorldApi`]' \
    --output table

# Tail Lambda function Logs using Logical name defined in SAM Template
sam logs -n HelloWorldFunction --stack-name aws-sam-rest-api --tail
```

**NOTE**: Alternatively this could be part of package.json scripts section.
