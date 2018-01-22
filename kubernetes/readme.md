# Docker Deployment to Kubernetes (AKS) using VSTS

## Overview

This lab shows how to build custom images of ASP.NETCORE web application and deploy to **Azure Container Service (AKS)** using Visual Studio Team Services. These services run in a high-availability environment, patched and supported, allowing you to focus on your solution instead of the environment they run in.

[**Azure Container Service (AKS)**](https://azure.microsoft.com/en-us/services/container-service/) is the quickest way to use Kubernetes on Azure. This new service provides an Azure-hosted control plane, automated upgrades, self-healing, easy scaling, and a simple user experience for both developers and cluster operators. With AKS, customers get the benefits of open source Kubernetes without the complexity and operational overhead. Using Visual Studio Team Services helps create your application container images for faster deployments reliably by setting up a continuous build.

In this lab you will perform the following:

- Create an Azure Container Registry (ACR), AKS and Azure SQL server
- Provision VSTS Team Project with .NET Core application using [VSTS Demo Generator](https://vstsdemogenerator.azurewebsites.net/) tool
- Configure endpoints in VSTS to access Azure and AKS
- Database deployment and configure Continuous Deployment (CD) in VSTS
- Modify connection string & ACR configuration in the source code
- Initiate the build to automatically deploy the application

The below diagram details the VSTS DevOps workflow with Azure Container Service with AKS:

<a href="https://azure.microsoft.com/en-in/solutions/architecture/continuous-integration-deployment-containers/" target="_blank">

![](images/vstsaksdevops.png) </a>

- Firstly the source code changes are committed to the VSTS git repository
- VSTS will build the Docker image **myhealth.web** and push the image tagged with the build ID to the ACR. Subsequently it will publish the [Kubernetes deployment YAML file](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) as a build artifact
- VSTS will deploy **mhc-front** and **mhc-back** services into  the Kubernetes cluster using the YAML file. **mhc-front** is the application hosted on a load balancer whereas **mhc-back** is the [Redis](https://redis.io/) cache
- The Kubernetes cluster will then pull the **myhealth.web** image from the ACR into the [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/) and complete the rest of the deployment file instructions
- The myhealth.web application will be accessible through a browser once the deployment is successfully completed

Below are the descriptions for the terminologies used in the lab document to help you get started:

[**Kubernetes**](): 

[**Cluster**](): 

[**Images**](): 

[**Docker**](): 

[**Containers**](): 

[**Pods**](https://kubernetes.io/docs/concepts/workloads/pods/pod/): A Pod is the basic building block of Kubernetes–the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents a running process on your cluster.

[**Services**](https://kubernetes.io/docs/concepts/services-networking/service/): A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service.

[**Deployments**](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/): A Deployment controller provides declarative updates for Pods

[**Kubernetes Manifest file**](https://kubernetes.io/docs/reference/kubectl/cheatsheet/): Kubernetes manifests with deployments, services and pods can be defined in json or yaml. The file extension .yaml, .yml, and .json can be used.

## Prerequisites

1. **Microsoft Azure Account**: You need a valid and active azure account for the labs.

1. Spin up a [Windows virtual machine on Azure](https://portal.azure.com/#create/Microsoft.WindowsServer2016Datacenter-ARM).

1. **Visual Studio Team Services Account**: If you don’t have one, you can create from [here](https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate)

1. You will need [Personal Access Token](https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate) for VSTS

1. You need to install **Kubernetes extension** from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=tsuyoshiushio.k8s-endpoint)  to your VSTS account

Follow the below steps for configuration using Windows Azure virtual machine (VM).

1. Install [Azure CLI version 2.0.23](https://azurecliprod.blob.core.windows.net/msi/azure-cli-2.0.23.msi)

1. Install [KubeCtl](https://kubernetes.io/docs/tasks/tools/install-kubectl/), and make sure kubectl is added to [PATH Environment Variable](https://msdn.microsoft.com/en-us/library/office/ee537574(v=office.14).aspx)

1. Create public & private [SSH RSA keys](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/ssh-from-windows). The contents of the public key **id_rsa.pub** is required for setting up environment

1. You need [Azure Service Principal Client ID and Client Secret](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)

## Setting up the Environment

We require below azure resources for this lab:

  <table width="100%">
    <thead>
      <tr>
         <th width="50%"><b>Azure resources</b></th>
         <th><b>Description</b></th>
      </tr>
    </thead>
    <tr>
      <td><img src="images/container_registry.png" width="30px"><b>Azure Container Registry</b></td>
      <td>Used to store images privately</td>
    </tr>
    <tr>
      <td><img src="images/aks.png" width="30px"> <b>AKS</b></td>
      <td>Docker images are deployed to Pods running inside AKS.</td>
    </tr>
    <tr>
      <td><img src="images/sqlserver.png" width="30px"> <b>SQL Server</b> </td>
      <td>SQL Server to host database</td>
    </tr>
    </table>

1. Click on **Deploy to Azure** (or right click and select ***Open in new tab***) to spin up **Azure Container Registry**, **Azure Container Service (AKS)** and **Azure SQL Server**. Enter required details and agree to ***Terms and Conditions***, and click **Purchase**.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVSTS-DevOps-Labs%2Fkubernetes%2Fkubernetes%2Ftemplates%2Fazuredeploy.json" target="_blank">

   ![](http://azuredeploy.net/deploybutton.png)</a>

   **Note**: Use lower case for ***DB Server Name***. Click [here](https://azure.microsoft.com/en-in/regions/services/) to to see Azure products available by region.

   ![](images/customtemplate1.png)
   ![](images/customtemplate2.png)

1. It takes approximately 5 minutes to provision the environment. Click on **Go to resource group**.

   ![](images/deploymentsucceeded.png)

1. Below components are created after deployment.

   ![](images/azurecomponents.png)

1. Click on **mhcdb** SQL database. Note down the **Server name**.

   ![](images/getdbserverurl.png)

1. Go back to the resource group, click on container registry and note down the **Login server** name.

    ![](images/getacrserver.png)

1. Switch back to the resource group. Click on your container service and note down the **API server address**. We need these details later in Exercise 2.

   ![](images/getaksserver.png)

   Now that required the azure components are created, let us create a VSTS project.

## Setting up the VSTS Project

1. Use [VSTS Demo Generator](https://vstsdemogenerator.azurewebsites.net/?TemplateId=77372&name=AKS) to provision the project on your VSTS account.

    ![](images/vstsdg.png)

1. Provide the Project Name, and click on Create Project.

   ![](images/vstsdemogen2.png)

1. Once the project is provisioned, click the **URL** to navigate to the project.

   ![](images/vstsdemogen3.png) 

## Exercise 1: Endpoint Creation

Since the connections are not established during project provisioning, let us manually create the Azure and Kubernetes endpoints.

1. In VSTS, navigate to **Services** by clicking on the gear icon ![](images/gear.png), and click on **+ New Service Endpoint**. Select **Azure Resource Manager**. Specify **Connection name**, select your **Subscription** from the dropdown and click **OK**. We use this endpoint to connect **VSTS** and **Azure**.

   ![](images/azureendpoint.png)

    You will be prompted to authorize this connection with Azure credentials. Disable pop-up blocker in your browser if you see a blank screen after clicking OK, and retry the step.

1. Click **+ New Service Endpoint**, and select **Kubernetes** from the list. We use this endpoint to connect **VSTS** and **Azure Container Service (AKS)**.

    For **Server URL**, enter your container service **API server address** pre-fixed with **http://**

    To get **Kubeconfig** contents, run these commands from your Azure CLI.

    - **az login**

      Authorize your login by going to below url, and enter the provided unique code.

      ![](images/azlogin.png)

    - **az aks get-credentials --resource-group yourResourceGroup --name yourAKSname**

      ![](images/getkubeconfig.png)

    - Navigate to **.kube** folder under your home directory (eg: C:\Users\YOUR_HOMEDIR\ .kube)

    - Copy contents from configuration file called **config** and paste it in the Kubernetes Connection window. Click **OK**.

      ![](images/aksendpoint.png)

## Exercise 2: Configure CI-CD

  Now that the connection is established, we will manually map the Azure endpoint, AKS and Azure Container Registry to build and release definitions.

 >Note : If you encounter an error - ***TFS.WebApi.Exception: Page not found*** for Azure tasks in the build/ release definition, you can fix this by typing a random text in the Azure Subscription field and click the **Refresh** icon next to it. Once the field is refreshed, you can select the endpoint from the drop down. This is due to a recent change in the VSTS Release Management API. We are working on updating VSTS Demo Generator to resolve this issue.

1. Go to **Builds** under **Build and Release** tab, **Edit** the build definition **MyHealth.AKS.Build**.

   ![](images/build.png)

1. In the **Process** section, select endpoint components from the dropdown under **Azure subscription** and **Azure Container Registry** as shown. Click **Save** under **Save & queue**.

    ![](images/updateprocessbd.png) 

    |Tasks|Usage|
    |-----|-----|
    |![](images/icon.png) **Run services**| prepares suitable environment by restoring required packages|
    |![](images/icon.png) **Build services**| builds images specified in a **docker-compose.yml** file with registry-qualified names and additional tags such as **$(Build.BuildId)**|
    |![](images/icon.png) **Push services**| pushes images specified in a **docker-compose.yml** file, to container registry|
    |![](images/publish-build-artifacts.png) **Publish Build Artifacts**| publishes the myhealth.dacpac file to VSTS|


1. Go to **Releases** under **Build & Release** tab, **Edit** the release definition **MyHealth.AKS.Release** and select **Tasks**.

   ![](images/release.png) 

   ![](images/releasetasks.png) 

1. Under **Execute Azure SQL: DacpacTask**, update **Azure Subscription** from the dropdown.

    ![](images/update_CD3.png) 

1. Under **Create Deployments & Services in AKS** task, update **Kubernetes Service Connection**. Expand **Container Registry Details** section and update **Azure subscription** and  **Azure Container Registry** with the endpoint components from the dropdown. Repeat similar steps for **Update image in AKS** task.

    ![](images/update_rd1.png) 


    - **Create Deployments & Services in AKS** will create deployments and services in AKS as per configuration specified in **mhc-aks.yaml** file. Pods will pull the latest image for first time.

    - **Update image in AKS** will pull the appropriate image corresponding to the BuildID from repository specified, and deploys the image to **mhc-front pod** running in AKS.

1. Click on **Variables** section, update **ACR** and **SQL server** values with the details noted earlier while setting up the environment. Click **Save**.

   >Note: **Database Name** is set to **mhcdb**, **Server Admin Login** is **sqladmin** and **Password** is **P2ssw0rd1234**.

   ![](images/releasevariables.png)

## Exercise 3: Update Connection String & ACR in Code

We will update the connection string in .NET Core application, and update ACR in manifest YAML file.

1. Go to **Code** tab, and navigate to the below path to **edit** the file **appsettings.json**

   >AKS/src/MyHealth.Web/**appsettings.json**

    Go to line number **9**. Provide the database server name as given in the step 6 of the previous exercise and manually update the **User ID** to **sqladmin** and **Password** to **P2ssw0rd1234**. Click **Commit**.
    
    > "DefaultConnection": "Server=YOUR_SQLSERVER_NAME.database.windows.net,1433;Database=mhcdb;Persist Security Info=False;User ID=sqladmin;Password=P2ssw0rd1234"

   ![](images/pasteconnectionstring.png) 

1. Navigate to the below path to **edit** the file **mhc-aks.yaml**.

    >AKS/**mhc-aks.yaml**

    Go to line number **93**. Update **YOUR_ACR** with your **ACR Login server** which was noted earlier while setting up the environment. Click **Commit**.

    ![](images/editmhcaks.png) 

     This YAML manifest file contains configuration details of **deployments**, **services** and **pods** which will be deployed in Kubernetes.


## Exercise 4: Trigger a Build and deploy application

In this exercise, we will trigger a build which in turn triggers an automatic deployment of the application. Our application is designed to be deployed in the pod with **load balancer** in front-end and **Redis cache** in the back-end.

1. Go to **Builds** under **Build and Release** tab, click the build definition **MyHealth.AKS.Build** and click **Queue new build...**. 

    ![](images/manualbuild.png) 

1. Go to **Builds** tab. Click on the build number to see the build in progress.

    ![](images/clickbuild.png)  

    ![](images/buildinprog1.png) 

1. The build will generate and push the image to ACR. After build completes, you will see the build summary.
To see the generated images in Azure Portal, go to **Azure Container Registry** and navigate to **Repositories**.

    ![](images/imagesinrepo.png)

1. Switch back to VSTS. Go to **Releases** tab, and double click on latest release. Go to **logs** to see the release summary.

    ![](images/releaseinprog.png)

    ![](images/release_summary1.png)

1. Once the release is complete, go to commandline and run below command to see the pods running in AKS:

    >**kubectl get pods**

    ![](images/getpods.png)

    Our web application is running in these pods.

1. To access your application, run the below command. If you see that **External-IP** is pending, wait for a while until an IP is assigned.

    >**kubectl get service mhc-front --watch**

    ![](images/watchfront.png)

1. Copy **External-IP** and paste in your browser to see the changes.

    ![](images/finalresult.png)

    **To access AKS through browser:**

    >**az aks browse --resource-group yourResourceGroup --name yourAKSname**

    ![](images/aksbrowse.png)

    </br>

    **AKS Dashboard:**

    ![](images/aksdashboard.png)


## Summary

AKS reduces the complexity and operational overhead of managing a Kubernetes cluster by offloading much of that responsibility to Azure. With **Visual Studio Team Services** and **Azure Container Services (AKS)**, we can build DevOps for dockerized applications by leveraging docker capabilities enabled on VSTS Hosted Agents.

## Feedback

Please let [us](mailto:devopsdemos@microsoft.com) know if you have any feedback on this lab.