# AWS CDK Fundamentals

## Useful commands

 * `npm run build`           compile typescript to js
 * `npm run watch`           watch for changes and compile (must run this in the background in another terminal window)
 * `npm run test`            perform the jest unit tests
 * `cdk bootstrap`           install a bootstrap stack for the first time you deploy an AWS CDK app into an environment or update existing bootstrap stack
 * `cdk synth`               emits the synthesized CloudFormation template in the cdk.out directory
 * `cdk diff`                compare deployed stack with current state. safe way to check what will happen once we run `cdk deploy`
 * `cdk deploy`              deploy this stack to your default AWS account/region. Use as the "build" step for production
 * `cdk deploy --hotswap`    will assess whether a hotswap deployment can be performed instead of a CloudFormation deployment. If possible, the CDK CLI will use AWS service APIs to directly make the changes; otherwise it will fall back to performing a full CloudFormation deployment. This is a one-time operation. Don't use for production.
 * `cdk watch`               monitors your code and assets for changes and attempts to perform a deployment automatically when a change is detected. Defined in cdk.json, it observes the files defined in the `include` array except those defined in the `exclude` array. Use this for faster development
 * `cdk ls`                   list all the stacks in an AWS CDK app
 * *Note: this project was bootstrapped with `cdk init sample-app --language typescript`*

## Concepts

#### [Constructs](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html)
 * Are the basic building block of CDK apps. They're like components in React.
 * Like React components, constructs can be made up of other contructs. However in the end, they all boil down to individual AWS services
 * `CdkWorkshopStack`, `lambda.Function`, `HitCounter`, `apigw.LambdaRestApi`, and `TableViewer` are all constructs
 * All share the signature (scope, id, props)
 * **`scope`**: the first argument is always the scope in which this construct is created. In almost all cases, you’ll be defining constructs within the scope of current construct, which means you’ll usually just want to pass `this` for the first argument
 * **`id`**: the second argument is the local identity of the construct. It’s an ID that has to be unique amongst construct within the same scope. The CDK uses this identity to calculate the CloudFormation Logical ID
 * **`props`**: the last (sometimes optional) argument is always a set of initialization properties. Those are specific to each construct
#### [Stacks](https://docs.aws.amazon.com/cdk/v2/guide/stacks.html)
 * Unit of deployment
 * Implemented through AWS CloudFormation stacks
#### [App](https://docs.aws.amazon.com/cdk/v2/guide/apps.html)
 * We call your CDK application an app, which is represented by the AWS CDK class `App`
 * Is the entrypoint of the CDK application
 * Can mount multiple stacks

## Testing

#### [Fine-Grained Assertion Tests](https://docs.aws.amazon.com/cdk/v2/guide/testing.html#testing_fine_grained)
 * Are like unit tests

## Pipelines

* Every pipeline requires at bare minimum:
* `synth(...)`: The `synthAction` of the pipeline describes the commands necessary to install dependencies, build, and synth the CDK application from source. This should always end in a synth command, for NPM-based projects this is always `npx cdk synth`
* The `input` of the synth step specifies the repository where the CDK source code is stored.
* CDK Pipelines auto-update for each commit in a source repo
* _Note: I created a local branch aws-origin that is set up to track remote branch 'main' from 'aws-origin' in order to push changes to AWS CodeCommit_

## Useful References

 * Official [AWS CDK v2 Construct Library](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html)
 * [Open-Source CDK Community](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=2&sort=downloadsDesc&offset=0)
 * [Testing](https://docs.aws.amazon.com/cdk/v2/guide/testing.html)

# AWS CDK TypeScript Workshop

## Architecture Diagram of this repo
![Architecture diagram of AWS services described below](readme-images/architecture-diagram.png)

## The instance of the stack (`CdkWorkshopStack`) contains the following:

 * [Cloudformation](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/cloudformation.png) which bundles together all the AWS services below
 * [Simple Lambda function](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/lambda-hello.png) which outputs messages
 * [Proxied Lambda function](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/lambda-hitcounter.png). Any request to any URL path will be proxied directly to our Lambda function, and the response from the function will be returned back to the user.
 * [API Gateway](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/api-gateway.png). Proxies all requests to the Lambda function above
 * [DynamoDB](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/dynamoDb.png) table that keeps track of how many requests were recieved per endpoint
 * [EC2](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/ec2.png) autogenerated
 * [cdk-dynamo-table-viewer](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/lambda-table-viewer.png) construct library installed via [npm](https://www.npmjs.com/package/cdk-dynamo-table-viewer)

## The instance of the stack (`CdkWorkshopPipelineStack`) contains the following:
 * [CodeCommit](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/codecommit-workshoprepo.png). Stores the code in AWS source control
 * [CodePipeline](https://github.com/JacobGrisham/aws-cdk/blob/main/readme-images/codepipeline-workshopipeline.png) Continuous Delivery, no need to run `cdk deploy` anymore! Just commit and push your code and the pipeline will deploy the production code for us

## Folder and File Structure

Generally, the structure can be summarized as follows:
 * the file in the bin folder is the entrypoint of the CDK application.
 * the files in the lib folder (sometimes labeled as src) is where the AWS resources are defined
 * within the lib (or src) folder are the stacks and this is where the AWS resources are called/instatiated/created.
 * the files in the lambda folder are the business logic

Here is a deeper dive into what each file contains in this application:
 * [bin/cdk-workshop.ts](https://github.com/JacobGrisham/aws-cdk/blob/main/bin/cdk-workshop.ts) is where `App` is instantiated and where it will load the stack defined in [lib/cdk-workshop-stack.ts](https://github.com/JacobGrisham/aws-cdk/blob/main/lib/cdk-workshop-stack.ts)
 * [lib/cdk-workshop-stack.ts](https://github.com/JacobGrisham/aws-cdk/blob/main/lib/cdk-workshop-stack.ts) is where your CDK application’s main stack is defined.
 * [lib/hitcounter.ts](https://github.com/JacobGrisham/aws-cdk/blob/main/lib/hitcounter.ts) is where the hit counter construct is defined
 * [lambda/hello.js](https://github.com/JacobGrisham/aws-cdk/blob/main/lambda/hello.js) is where the logic for displaying messages is defined
 * [lambda/hitcounter.js](https://github.com/JacobGrisham/aws-cdk/blob/main/lambda/hitcounter.js) is where the logic for handling hits is defined
 * [lib/pipeline-stage.ts](https://github.com/JacobGrisham/aws-cdk/blob/main/lambda/pipeline-stage.ts) declares a new Stage (component of a pipeline) and in that stage instantiates our application stack
 * [lib/pipeline-stack.ts](https://github.com/JacobGrisham/aws-cdk/blob/main/lambda/pipeline-stack.ts) `const deploy` imports and creates an instance of the WorkshopPipelineStage. Later, you might instantiate this stage multiple times (e.g. you want a Production deployment and a separate devlopment/test deployment).
 * [cdk.json](https://github.com/JacobGrisham/aws-cdk/blob/main/cdk.json) file tells the CDK Toolkit how to execute your app. In our case it will be `npx ts-node bin/cdk-workshop.ts`. More specifically, it contains definitions for `cdk` commands.
 * _Note that educational code comments are signified with "👇" and "👈". These are comments that I wouldn't normally include in production code, but since the purpose of this repo is for learning and education, I've included them to explain very fine details.

## Results

 * You can hit the API gateway endpoint on your browser here [https://46rfkzmiqg.execute-api.us-west-1.amazonaws.com/prod/](https://46rfkzmiqg.execute-api.us-west-1.amazonaws.com/prod/) or via the CLI with `curl -i https://46rfkzmiqg.execute-api.us-west-1.amazonaws.com/prod/`
 * If using the CLI, you'll see a response message in the same terminal that you initiated the command
 * Add anything after [prod/](https://46rfkzmiqg.execute-api.us-west-1.amazonaws.com/prod/) like [prod/some-random-endpoint](https://46rfkzmiqg.execute-api.us-west-1.amazonaws.com/prod/some-random-endpoint)
 * And view the DynamoDB table with the number of hits per endpoint here [https://k5n66tmta3.execute-api.us-west-1.amazonaws.com/prod/](https://k5n66tmta3.execute-api.us-west-1.amazonaws.com/prod/). Here's an example of what it looks like ![cdk-dynamo-table-viewer](/readme-images/cdk-dynamo-table-viewer.png)
 * Running `npm run test` should output the following:
 ```
  PASS  test/hitcounter.test.ts
  ✓ DynamoDB Table Created (171ms)
  ✓ Lambda Has Environment Variables (52ms)
  ✓ Read capacity can be configured (47ms)

  Test Suites: 1 passed, 1 total
  Tests:       3 passed, 3 total
  Snapshots:   0 total
  Time:        3.913s
  Ran all test suites.
 ```
