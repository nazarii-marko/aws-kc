## GoodFish setup instructions

`cd ./path/to/project`  
`yarn` to install dependencies  
`cp .env.example .env` & update the configs  
`sudo yarn serverless deploy` to generate cloudFormation stack, bundle the app and deploy to AWS

After this procedure, a new cloudFormation stack will appear in AWS. Name is **ServiceName-StageName** (e.g. GoodFish-dev). If you select this stack & navigate to _Resources_ tab, you will be able to view a list of all created resources. Currently we are interested in VPC:
![VPC](/instructionsImages/vpc.png 'VPC')  
Now, we need to create a VPC Endpoint, in order to access a public S3 bucket from the VPC:  
![endpoint1](/instructionsImages/endpoint1.png 'endpoint1')  
![endpoint2](/instructionsImages/endpoint2.png 'endpoint2')  
It has to be for service **com.amazonaws.eu-central-1.s3** and of type **Gateway**. VPC - the one from CloudFormation stack & Route Table - the one that's prepopulated:  
![endpoint3](/instructionsImages/endpoint3.png 'endpoint3')  
Nothing else needs to be changed.  
Then, we will need to update the DB schema & populate the database with some data. For that, there are 3 Lambdas:

- GoodFish-dev-seed
- GoodFish-dev-seedAdmins
- GoodFish-dev-seedConstituents
- GoodFish-dev-migrate

The reason for splitting them into separate functions is the time limits for execution of Lambda.  
If we go to the Lambda's page and select **Test** tab, we will be able to trigger the function manually:
![lambda1](/instructionsImages/lambda1.png 'lambda1')  
Also, **seed** and **seedConstituents** accept inputs. Therefore, prior to executing them, we will need to adjust the event values:  
![lambda2](/instructionsImages/lambda2.png 'lambda2')

### Order of triggering functions:

- **migrate**
- **seed**, _{ "batch": 1 }_
- **seed**, _{ "batch": 2 }_
- **seed**, _{ "batch": 3 }_
- **seedAdmins**
- **seedConstituents**, _{ "batch": 1 }_
- **seedConstituents**, _{ "batch": 2 }_
- **seedConstituents**, _{ "batch": 3 }_
- **seedConstituents**, _{ "batch": 4 }_

Again, this was splitted due to the limitations of Lambda and amount of data we need to seed.

After deploying backend to Lambda, we will see output similar to this:
![deployment](/instructionsImages/deployment.png 'deployment')  
We will need address of **graphql** endpoint. In Front-End repository, we need to create a .env file and add API*HOST to it.  
For example, `API_HOST=https://1cqfbzlme9.execute-api.eu-central-1.amazonaws.com`.  
*! Please note that only host is required, without endpoint or trailing slash\_.
Then, we need to run `yarn && yarn nuxt build && yarn nuxt generate`. This will generate a `dist` folder in the root directory of a project. Now, if we go to AWS Amplify app and click `Host your web app`, we have may options for integration with different version control systems. For the time being, let's deploy without git provider:  
![amplify](/instructionsImages/amplify.png 'amplify')  
![amplify2](/instructionsImages/amplify2.png 'amplify2')  
This step will give similar output to this:  
![amplify3](/instructionsImages/amplify3.png 'amplify3')  
The final step would be to update `.env` with the domain in backend repository and redeploy it with `sudo yarn serverless deploy`.
