# Azure CI/CD Pipelines for UI 

## What is CI CD process in DevOps?

*CI* stands for continuous integration, a fundamental DevOps best practice where developers frequently merge code changes into a central repository where automated builds and tests run. But *CD* can either mean continuous delivery or continuous deployment

## About this exercise

In this lab we will be working on two code Bases, **Backend Code base** and **Frontend Code Base**. 

Previously we developed a base structure of an api solution in Asp.net core that have just two Api functions GetLast12MonthBalances & GetLast12MonthBalances/{userId} which returns data of the last 12 months total balances.
Later on we created a proper CI/CD Pipeline for our Api. Now our API is running at this URL for test environment https://bbbankapitest.azurewebsites.net/api/Transaction/GetLast12MonthBalances/

and at this URL for production environment 
https://bbbankapiprod.azurewebsites.net/api/Transaction/GetLast12MonthBalances/

![](/BBBank_UI/src/assets/images/5.png)

Learn more about [API CI/CD Pipelines](https://github.com/PatternsTechGit/PT_CI-CD_Azure_Pipelines_for_API) 


For more details about this base project See: https://github.com/PatternsTechGit/PT_ServiceOrientedArchitecture

------------------
## Frontend Code base:

Previously we scaffolded a new Angular application in which we have integrated

- FontAwesome
- Bootstrap toolbar
- Returned data in html Table from a HTTP call we generated. 

Our recent output result at localhost looks like this
![](/BBBank_UI/src/assets/images/6.png)

---------------------

## In this exercise
- We will create Azure Static App
- We will create Continuos Integrated CI Build Pipeline
- We will set a stage ( necessary permissions) before release pipeline.
- We will create Continuous Deployment CD Release Pipeline
- We will make changes in API and UI codes
---------------

## Step 1: Creating Static Website in Azure

There are several ways to create a static website in Azure. In this lab I will explain a way to create Free static website in a storage account.

- Open Azure Portal, search *Storage Account* and hit *Create* button. 
![](/BBBank_UI/src/assets/images/7.png)

- We will select our resource group where all our resources should be
- Give a meaningful name to your storage account 
- Keep remaining items as default and hit *Review and Create* button.
![](/BBBank_UI/src/assets/images/8.png)

- Create another storage account for Production environment in the same way.
- Now we have 2 storage accounts in which we will create 2 different static website for 2 different scenarios.
![](/BBBank_UI/src/assets/images/9.png)

- Now first we will work on test environment. 
- Select test storage account we created
- Scroll down and select *Static Website*
- Enable the static website
- Give index document name. In our case it will be *index.html*
- Save this configuration
![](/BBBank_UI/src/assets/images/10.png)

- It will give you a default URL at which your website is hosted
![](/BBBank_UI/src/assets/images/13.png)

- Now if you scroll up and select **Containers** you will see $web file in it.
- Now if you run `npm build` command in your code editor terminal it will create a folder named *dist*
- If we manually upload all the contents of this folder into *$web* folder you will see your website running at provided URL. 
![](/BBBank_UI/src/assets/images/12.png)

- But in this lab we want to automate and this task will be done by our CD pipeline.
- At the moment our $web folder is empty
- Follow the same procedures for **Production Storage Account** to create another website for production environment. 

------------------

## Step 2: Creating Continuous Integrated CI Pipeline

- Now we will start our procedure to create a BUILD pipeline. 
- Open Azure Devops portal, organization and the project you are working on.
- Select *pipelines* from the left menu
![](/BBBank_UI/src/assets/images/14.png)

- Click *New Pipeline* button. 
- We will use classic editor for this lab
![](/BBBank_UI/src/assets/images/15.png)

- Select github as a source and connect your github to your Azure pipeline via service connections 
![](/BBBank_UI/src/assets/images/16.png)

- Use OAuth or personal access token from github to connect you account. [Follow this link](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to get personal access token from github. 
![](/BBBank_UI/src/assets/images/17.png)

- Select your github repository and branch you want to authorize it to. 
![](/BBBank_UI/src/assets/images/19.png)
- Click create tab

- We will create an empty job at the moment and will add tasks to it later on
![](/BBBank_UI/src/assets/images/20.png)

- Give a meaningful name to your pipeline 
- Select your source by which this job will be triggered 
- Clean all the residuals before running any task
![](/BBBank_UI/src/assets/images/21.png)

- Now we will add out first task.
- Our first task will be to install node package module
- To fulfill this task we will click '+' tab from the agents job
- Then search npm and add it to tasks
![](/BBBank_UI/src/assets/images/22.png)

- We will be installing the node module so we will use install command
- Then give the path where your package.json file is located.
![](/BBBank_UI/src/assets/images/23.png)

- Now we will use the build step
- Follow the same procedure and add another npm task
- This time name it accordingly. We will name it npm build
- We will use custom command in it
- Give path of you package.json folder
- Write the npm custom command. In out case it will *run build* and it will run `npm run build` 
- Save your task
![](/BBBank_UI/src/assets/images/24.png)

- Now we will publish this build artifact 
- For this add another task.
- Search publish artifact and add *publish build artifact*
![](/BBBank_UI/src/assets/images/25.png)

- Give meaningful name to this task
- Give the location where the folder will be dropped
- Give name to the folder. In our case name will be *dist*
![](/BBBank_UI/src/assets/images/26.png)

- In the end make sure your automattic trigger in enabled 
- Enabling it will automatically start this pipeline whenever there is a change in github repository.
- Save and run your pipeline
![](/BBBank_UI/src/assets/images/27.png)

- After running the pipeline it will follow all the steps and give this output if successful
- Here you can find the location of artifact
- We will publish this artifact in static website via CD pipeline
![](/BBBank_UI/src/assets/images/31.png)


-----------------
## Step 3: Giving permission to organization to make changes in our Static Website

- Before moving on to CD pipeline we will first make changes in CI pipeline
- Open Azure Portal and go to the static website we created
- In left menu go to *Access Control (IAM)* and add role assignment
![](/BBBank_UI/src/assets/images/28.png)

- First we will give Owner role to our organization
- Select Owner tab, click 'select member', search your organization name and save it
![](/BBBank_UI/src/assets/images/29.png)

- Now we will assign *Storage blob data owner* role in the same way
![](/BBBank_UI/src/assets/images/30.png)

- Follow the same procedures for Production storage account

-------------------------

## Step 4: Creating CD pipeline

- Now we will take our artifact from drop location of CI pipeline and publish it to Static webapp $web location
- To do this go to release pipeline from the left menu and create new pipeline

 ![](/BBBank_UI/src/assets/images/32.png)

 - Select Empty job so we can add tasks to it later
 - Give meaningful name to this stage and save it
 ![](/BBBank_UI/src/assets/images/33.png)

 - Now we add 2 tasks for this stage
 - First we will delete all the files from $web location so there will be no residual in it
 - Then we will publish our new artifact in $web location
 - To delete existing files from location, search Azure CLI and add it
 - We will run this powershell script to delete this file
 ```powershell
 az storage blob delete-batch --source '$web' --account-name bbbankuitest
 ```
- In Azure Cli task select your subscription
- Our script will inline Powershell script
- We will run the above given script with our storage account name in the end
![](/BBBank_UI/src/assets/images/34.png)

- This will delete all the files from given location ($web) in static website

- Now we will add another task to publish our artifact in static website
- For this you need to add another task named *Azure file copy*
![](/BBBank_UI/src/assets/images/35.png)

- Give name to this task
- Give the exact location of artifact which we dropped in CI pipeline
- Select the subscription in which our storage account is
- Write exact name of your storage account and container 
![](/BBBank_UI/src/assets/images/36.png)

- In last process enable continuous trigger from this tab
- Enabling it will automatically run this pipeline whenever there is new file in drop location
![](/BBBank_UI/src/assets/images/37.png)

It will successfully create you CD pipeline. You can run this manually to check its functionality 
![](/BBBank_UI/src/assets/images/38.png)





