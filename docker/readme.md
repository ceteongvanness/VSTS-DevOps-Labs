## Docker Deployment using VSTS and Azure for ASP.NETCORE

## Overview

This lab shows how to build custom images of <a href="http://dockr.ly/2zLiDPy">**Dockerized ASP.NETCORE**</a> application, push those images to <a href="http://dockr.ly/2AJLgge"> **Private Repository** </a> (<a href="http://bit.ly/2jssVQy"> Azure Container Registry </a>), and pull these images to deploy to containers in **Azure Web App** (Linux) using Visual Studio Team Services.

Web App for containers lets you bring your own <a href="http://dockr.ly/2imRbR4">Docker</a> formatted container images, easily deploy and run them at scale with Azure. Combination of Team Services and Azure integration with Docker will enable you to:

1.  <a href="http://dockr.ly/2z2Qsi2"> Build </a> your own custom images using <a href="http://bit.ly/2jqGujv"> VSTS Hosted Linux agent </a>
2. <a href="http://dockr.ly/2hAZco0"> Push </a> and store images in your private repository
3. Deploy and  <a href="http://dockr.ly/2AJPaEW"> run </a> images inside containers

Below screenshot helps you understand the VSTS DevOps workflow with Docker: 

<img src="images/vstsdockerdevops.png">


## Pre-requisites

1.  **Microsoft Azure Account**: You need a valid and active azure account for the labs.

2. You need a **Visual Studio Team Services Account** and <a href="http://bit.ly/2gBL4r4">Personal Access Token</a>.

    <img src="images/vstsdemogen.png">

3. You need to install **Docker Integration** extension from <a href="http://bit.ly/2hurgK3">Visual Studio Marketplace</a>.

## Setting up the Environment

We will create an **Azure Container Registry** to store the images generated during VSTS build. These images contain environment configuration details with build settings.  An **Azure Web App** (with Linux OS) is created where custom built images will be deployed to run inside containers. 

1. Click on **Deploy to Azure** to spin up **Azure Container Registry**, **Azure Web App** and **Azure SQL Database** along with **Azure SQL Server**.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVSTS-DevOps-Labs%2Fdocker%2Fdocker%2Ftemplates%2Fazuredeploy.json" target="_blank">

    <img src="http://azuredeploy.net/deploybutton.png"/>
    </a> 

   <br/>

   <img src="images/createacr-linuxwebapp.png">

2. It takes approximately **3 to 4 minutes** to provision the environment. Click on **Go To resource group**.

   <img src="images/acrdeploymentsucceeded.png">

3. Below components are created post deployment. Click on **Azure Container Registry**.

     
    <table width="100%">
     <thead>
      <tr>
         <th width="50%"><b>Azure Components</b></th>
         <th><b>Description</b></th>
      </tr>
    </thead>
    <tr>
      <td><img src="images/container_registry.png" width="30px"><a href="http://bit.ly/2mwVYUz"><b>Azure Container Registry</b></a></td>
      <td>Used to store images privately</td>
    </tr>
    <tr>
      <td><img src="images/storage.png" width="30px"> <a href="http://bit.ly/2iYiCQx"><b>Storage Account</b></a> </td>
      <td>Container Registry resides in this storage account</td>
    </tr>
    <tr>
      <td><img src="images/app_service.png" width="30px"> <a href="http://bit.ly/2ALhdES"><b>App Service</b></a>  </td>
      <td>Docker images are deployed to containers in this App Service</td>
    </tr>
    <tr>
      <td><img src="images/app_service_plan.png" width="30px"> <a href="http://bit.ly/2AINQ5x"><b>App Service Plan</b></a> </td>
      <td>Resource where App Service resides</td>
    </tr>
    <tr>
      <td><img src="images/sqlserver.png" width="30px"> <a href="http://bit.ly/2AINQ5x"><b>SQL Server</b></a> </td>
      <td>SQL Server to host database</td>
    </tr>
    <tr>
      <td><img src="images/sqldb.png" width="30px"> <a href="http://bit.ly/2AINQ5x"><b>SQL database</b></a> </td>
      <td>SQL database to host MyHealthClinic data</td>
    </tr>
    </table>

    </br>

    <img src="images/postazuredeployment.png">
    <img src="images/dbinazure.png">


4. Click on your container registry. Note down the **Login server** name. We need these details later in Excercise 2.

   <img src="images/acrloginserver.png">

## Setting up the Project

1. Use <a href="https://vstsdemogenerator.azurewebsites.net" target="_blank">VSTS Demo Data Generator</a> to provision a project on your VSTS account 

2. Select **Docker** for the template.
   <img src="">

3. Once the project is provisioned, select the URL to navigate to the project that you provisioned.

   <img src="">


## Exercise 1: Endpoint Creation

Since the connections are not established during project provisioning, we will manually create the endpoints. 

1. In VSTS, navigate to **Services** by clicking on the gear icon, and click on **+ New Service Endpoint**. Select **Azure Resource Manager**. Specify **Connection name**, select your **Subscription** from the dropdown and click **OK**. We use this endpoint to connect **VSTS** and **Azure**.

   <img src="images/armendpoint.png">

   You will be prompted to authorize this connection with Azure credentials. 

   **Note:** Disable pop-up blocker in your browser if you see a blank screen after clicking **OK**, and retry the step. 


## Exercise 2: Configure CI-CD

 Now that the connection is established, we will manually map the Azure endpoint and Azure Container Registry to build and release definitions.

1. Go to **Builds** under **Build and Release** tab, **Edit** the build definition **Docker**.

   <img src="images/build.png">

2. Click on **Process** section, select appropriate contents from dropdown under **Azure subscription** and **Azure Container Registry**.

   <img src="images/updateprocessbd.png">

   <br/>
   <br/>

   <table width="100%">
   <thead>
      <tr>
         <th width="50%"><b>Tasks</b></th>
         <th><b>Usage</b></th>
      </tr>
   </thead>
   <tr>
      <td><img src="images/icon.png"> <a href="http://bit.ly/2zlTspl"><b>Run services</b></a> </td>
      <td>prepares suitable environment by restoring required packages</td>
   </tr>
   <tr>
      <td><img src="images/icon.png"> <a href="http://bit.ly/2zlTspl"><b>Build services</b></a></td>
      <td>builds <b>service images</b> specified in a <b>docker-compose.yml</b> file with registry-qualified names and additional tags such as <b>$(Build.BuildId)</b></td>
   </tr>
    <tr>
      <td><img src="images/icon.png"> <a href="http://bit.ly/2zlTspl"><b>Push services</b></a></td>
      <td>pushes <b>service images</b> specified in a <b>docker-compose.yml</b> file, with multiple tags, to container registry</td>
   </tr>
    <tr>
      <td><img src="images/icon.png"> <a href="http://bit.ly/2zlTspl"><b>Lock services</b></a></td>
      <td>pulls image from default tag <b>latest</b> in container registry and verifies if uploaded image is up to date</td>
   </tr>
   <tr>
      <td><img src="images/copy-files.png"> <a href="http://bit.ly/2iDhjpO"><b>Copy Files</b></a> </td>
      <td>used to copy files from source to destination folder using match patterns </td>
   </tr>
   <tr>
      <td><img src="images/publish-build-artifacts.png"> <a href="http://bit.ly/2zGD6bn"><b>Publish Build Artifacts</b></a> </td>
      <td> used to share the build artifacts </td>
   </tr>
</table>
<br/>

3. Click **Save**.

   <img src="images/savebd.png">

4. Go to **Releases** under **Build & Release** tab, **Edit** the release definition **Docker** and select **Tasks**.

   <img src="images/release.png">

   <br/>

   <img src="images/release_tasks.png">

5. Update the **Azure Connection Type**, **Azure Subscription**, and SQL DB Details such as **Azure SQL Server Name**, **Database Name**, **Server Admin Login** and **Password**. Click **Save**.

    <img src="images/update_dbtask.png">

6. Trigger a release by clicking **+ Create Release** under **+ Release**. 

    <img src="images/createrelease.png">

    After this step is complete, the database schema will be deployed to SQL Database.

7. Right click on task **Execute Azure SQL : DacpacTask**, and select **Disable Selected Task(s)**. After this, right click on **Deploy Azure App Service** task, and select **Enable Selected Task(s)**.

<img src="images/disabletasks_rd.png">

</br>

<img src="images/enabletasks_rd.png">

8. Under **Deploy Azure App Service** task, update **Azure subscription** and **Azure Service name** with the endpoint components from the dropdown. In the **Registry or Namespace** field, enter **Azure Container Registry Login Server** from Azure portal. Let the image name be **myhealth.web**. Click **Save**.

   <img src="images/updatedrd.png">

**Deploy Azure App Service** will pull the appropriate image corresponding to the BuildID from repository specified, and deploys the image to Linux App Service. 

## Exercise 3: Update Connection String

1. Click on **Code** tab, and navigate to **healthclinic.sql** file under **Docker** repository. Copy entire content of this file.

    <img src="images/copysql.png">   

2. Switch to **Azure Portal**, and navigate to the **SQL Database** which you created at the beginning of this lab.Click on **Data Explorer**, and provide database **Password** to login. 

    <img src="images/dblogin.png">   

3. Under **Query** section, paste the content copied from **healthclinic.sql** file as shown, and click on **Run**. This will now push required data into the database, so that our sample application MyHealthClinic could interact with it.

    <img src="images/pastesql.png"> 

4.  Scroll down and select **Connection Strings** section. Copy the contents as shown.

    <img src="images/copy_connectionstring.png"> 

5. Switch to your VSTS account. Go to **Code** tab, and navigate to the below path to **edit** the file- 

    >Docker/src/MyHealth.Web/appsettings.json

    Go to line number **9**. Paste the connection string as shown and manually update the **User ID** and **Password**. Click on **Commit**.

   <img src="images/paste_connectionstring.png">


## Exercise 4: Code update

In this excercise, we will enable the continuous integration trigger to create a new build for each commit to the master branch, and update the code to trigger CI-CD. 

1. Go to **Builds** under **Build and Release** tab, **Edit** the build definition **Docker**.

   <img src="images/build.png">

2. Click on **Triggers** section, and check the option to **Enable continuous integration**. Click **Save**.

    <img src="images/enable_CI.png">

3. Go to **Code** tab, and navigate to the below path to edit the file- 

   >Docker/src/MyHealth.Web/Views/Home/**Index.cshtml**

   <img src="images/editcode.png">

4. Go to line number **28**, update **JOIN US** to **JOIN US TODAY**, and click **Commit**.

    <img src="images/lineedit.png">

5. Go to **Builds** tab to see the CI build in-progress.

    <img src="images/in_progress_build.png">

6. The build will generate and push the image to ACR. After build completes, you will see the build summary. 
    
    <img src="images/build_summary.png">
 
7.  Go to **Releases** tab to see the release summary with logs. The release will deploy the image to App Service based on the **BuildID**, which is tagged with the image.

    <img src="images/release_summary.png">

    <br/>

    <img src="images/release_logs.png">

8. Go to <a href="https://portal.azure.com">Azure Portal</a>, navigate to the **App Service** which was created at the beginning of this lab. Click on the **URL** to see the changes in your app.

    <img src="images/getwebappurl.png">

    <br/> 

    <img src="images/finalresult.png">

9. To see the generated images in Azure Portal, go to **Azure Container Registry** and navigate to **Repositories**.

    <img src="images/imagesinrepo.png">


## Summary

With **Visual Studio Team Services** and **Azure**, we can build DevOps for dockerized applications.

## Feedback

Please let <a href="mailto:devopsdemos@microsoft.com">us </a> know if you have any feedback on this lab.