Q1 - SCENARIO

A car rental company called FastCarz has a .net Web Application and Web API which are recently
migrated from on-premise system to Azure cloud using Azure Web App Service
and Web API Service.
The on-premises system had 3 environments Dev, QA and Prod.
The code repository was maintained in TFS and moved to Azure GIT now. The TFS has daily builds which
triggers every night which build the solution and copy the build package to drop folder.
deployments were done to the respective environment manually. The customer is planning to setup
Azure DevOps Pipeline service for below requirements:
1) The build should trigger as soon as anyone in the dev team checks in code to master branch.
2) There will be test projects which will create and maintained in the solution along the Web and API.
The trigger should build all the 3 projects - Web, API and test.
The build should not be successful if any test fails.
3) The deployment of code and artifacts should be automated to Dev environment.
4) Upon successful deployment to the Dev environment, deployment should be easily promoted to QA
and Prod through automated process.
5) The deployments to QA and Prod should be enabled with Approvals from approvers only.
Explain how each of the above the requirements will be met using Azure DevOps configuration.
Explain the steps with configuration details.

Step 1: The build should trigger as soon as anyone in the dev team checks in code to master branch.
  
  Add a trigger section in your Azure Pipeline which gets triggered on a commit change on a Master branch.
    Ex:   trigger:
           - master
           
Step 2: There will be test projects which will create and maintained in the solution along the Web and API.
           The trigger should build all the 3 projects - Web, API and test.
           
 - Using path filters select the projects which needs to be included in the pipeline
 - Using Stages -> Jobs -> Steps -> Tasks we can build and cancel it if any of the build stage fails using 'Conditions'

Step 3: The deployment of code and artifacts should be automated to Dev environment.

In the Stages we define the environment and get the artifacts from earlier build pipeline

Step 4: Upon successful deployment to the Dev environment, deployment should be easily promoted to QA
        and Prod through automated process.
        
 Use Release Pipelines for deploying to multiple environments
 
 Step 5: The deployments to QA and Prod should be enabled with Approvals from approvers only.
 
 Add 'Approvals' Task in the stages of Release pipeline
 
 ************************************** Pipeline Yaml *********************************
 
 
  trigger:
  branches:
    include:
      - master
  paths:
    include:
      - src/Connector      

pool:
  vmImage: ubuntu-latest

stages:
- stage: Tests
  jobs:
  - job: 
    displayName: Unit tests
    steps:
    - script: echo simulate running your unit tests!
      displayName: 'Run unit tests'
  - job: 
    displayName: UI tests
    steps:
    - script:  echo simulate running your ui tests!
      displayName: 'Run unit tests'

- stage: Build
  dependsOn: [] # This will remove implicit dependency and run in parallel with the stage: Tests above 
  jobs:
  - job:
    displayName: Build the application
    steps:
    - script: |
        echo Running all builds commands...
        echo ... commands successfully completed.
      displayName: 'Run build scripts & tasks'

- stage: Dev
  dependsOn:
  - Tests
  - Build
  jobs:
  - deployment:
    displayName: Dev deploy
    environment: Dev
    strategy:
     runOnce:
       deploy:
         steps:
           - script: echo Running in the Dev environment as deployment job
             displayName: 'Dev based stage'
             
- stage: Staging
  jobs:
  - deployment:
    displayName: Staging deploy
    environment: Staging
    strategy:
     runOnce:
       deploy:
         steps:
           - script: echo Running in the Staging environment as deployment job
             displayName: 'Staging based stage'

- stage: Production
  jobs:
  - deployment: 
    displayName: Production deploy
    environment: Production
    strategy:
     runOnce:
       deploy:
         steps:
           - script: echo Running in the Production environment as deployment job
             displayName: 'Production based stage'             




********* Approvals ********************


To add an approval in a YAML-pipeline, one needs to add an environment in Azure DevOps. Navigate to ‘Pipelines’ –> ‘Environments’. There you click on ‘New Environment’, you will see the following form. Add a name and leave the Resource section set to None. Click ‘Create’. 

Now it’s time to add the approval rule. You do that by clicking on the three stacked dots in the upper right corner and click ‘Approvals and Checks’.

There you click on ‘Approvals’ and will be directed to the following screen. There you enter the name of the person or group that needs to approve this stage. 
