# Working with Eclipse

VSTS helps teams modernize their application development lifecycle and go from idea to deployment with continuous integration, testing, and deployment for any app targeting any platform. VSTS works with (m)any development tool including Visual Studio, Eclipse, IntelliJ, Android Studio, XCode, etc., to make it easy for developers to use VSTS.

In this exercise, you are going to see a typical end-to-end workflow for a Java developer using VSTS and working with Eclipse. We will install and explore how **Team Explorer Everywhere** helps teams using Eclipse-based IDE to collobarate across teams with Visual Studio Team Services / Team Foundation Server. 

You should have completed the labs that set up automated build and release (this is a CI/CD pipeline). In this scenario, you will open the running MyShuttle application and discover a bug. You will then use the [Exploratory Testing extension](https://marketplace.visualstudio.com/items?itemName=ms.vss-exploratorytesting-web) to create a Bug work item in VSTS. You will then branch the code for fixing the bug. Once the bug is fixed on the branch, you will merge the code in via a Pull Request and code review. This will then automatically queue the build/release pipeline and your fix will be deployed.



## Pre-requisites

1.  **Microsoft Azure Account**: You need a valid and active azure account for the labs.

2. You need a **Visual Studio Team Services Account** and <a href="https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate">Personal Access Token</a>.


## Setting up the project

1. Use <a href="https://vstsdemogenerator.azurewebsites.net" target="_blank">VSTS Demo Data Generator</a> to provision a project on your VSTS account.

 ![VSTS Demo Generator](images/vstsdemogen.png)

1. Select **** for the template.

 ![VSTS Demo Generator](images/vstsdemogen.png)


3. Once the project is provisioned, select the URL to navigate to the project that you provisioned.

## Provisioning Eclipse VM on Azure

Click on **Deploy to Azure** to provision a Ubuntu VM pre-installed with Eclipse. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVSTS-DevOps-Labs%2Feclipse%2Feclipse%2Farm%2520template%2Fazuredeploy.json" target="_blank">
<img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Once the machine is provisioned, you can RDP to it. 

## Installing Team Explorer Everywhere


We wil install **Team Explorer Everywhere (TEE)**, the official plug-in for Eclipse from Microsoft to connect VSTS/TFS with Eclipse-based IDE on any platform. It is supported on Linux, Mac OS X, and Windows and is compatible with IDEs that are based on Eclipse 4.2 to 4.6. 

With Team Explorer Everywhere, you can:
* Browse and clone Git repositories
* Full access to TFS Version Control (TFVC), including check-in, check-out, sync, branch, merge, diff, etc.
* Full access to TFS agile tools, work items, and issue tracking capabilities allowing you to add, edit and query work items
* Full access to TFS Build functionality including the ability to create Ant, Maven, or Gradle based builds in TFS, publish JUnit test results into TFS or Visual Studio Team Services, monitor progress and handle results. This is fully compatible with all Team Foundation Build types including Gated Check-in and Continuous Integration Builds.

1.  Open Eclipse from the Launcher by clicking on the **Eclipse** icon from the panel below

1. Click **OK** to select the default folder for the workspace
The plugin can be acquired and installed in the following ways"

1. Eclipse MarketPlace:  The plug-in is freely available on the Eclipse Marketplace -https://marketplace.eclipse.org/content/team-explorer-everywhere. To install it from the marketplace, you can drag the **install** button to a running Eclipse or install it using the Help /|Eclipse Marketplace option

1. Eclipse download site:   
   1. Choose **Help** menu /| **Install New Software**
   1. Select **Add**
   1. In the **Add Repository** dialog, enter **Team Explorer Everywhere** for the name and specify **http://dl.microsoft.com/eclipse/** for the location
   1. Click **OK**
   1. This should populate the list of features in the install dialog box.  Select the check box that corresponds to the **Team Explorer Everywhere**
   1. Choose the **Next** button two times 
   1. The last page of the wizard shows you the **Microsoft Software License Terms**. Select the **Finish** button to accept if you agree with the terms

1. GitHub Repository:You can also manually download and install the plug-in from the Releases section of this [GitHub](https://github.com/Microsoft/team-explorer-everywhere) repository.

Restart Eclipse if required.

## Connecting to Visual Studio Team Services

The TEE plugin, when installed adds a new perspective **"Team Foundation Server Exploring"** and couple of views (windows) to access VSTS/TFS. 

1. Select **Window/| Perspective /| Open Perspective /| Other...** and choose **"Team Foundation Server Exploring"**. If you do not want to change *perspective*, select **Window /| Show View /| Team Foundation Server /| Team Explorer** 

1. On the **Team Explorer** window, select **Connect to Team Services or Team Foundation Server**

 ![](images2/teamexplorer-default.png)

 1. Select the checkbox to accept the **End User License Agreement**

 1. On the **Server Selection** page, select the **Connect to a Team Foundation Server or Team Services account** option and enter the URL of the team services
  ![](images2/serverselection.png)


Install the Exploratory Testing Extension for Chrome
----------------------------------------------------
In this task you will install the [Exploratory Testing extension](https://marketplace.visualstudio.com/items?itemName=ms.vss-exploratorytesting-web) into Chrome.

1. Open chrome and navigate to `https://chrome.google.com/webstore`. Enter "exploratory testing" into the search box. Find the "Test & Feedback" extension from Microsoft Corporation and click "Add to Chrome". Click Install in the dialog.

    
   <img src="images/add-ext.png">
    

1. Once installed, a beaker icon appears in the top right of the Chrome toolbar. Click it to open the UI.
1. Click on the gear icon to open the settings. Select "Connected" and enter your VSTS account URL and click Next.

    ![Connect to VSTS](images/e2e-eclipse/connect-to-vsts.png "Connect to VSTS")

1. Select your team project and expand it and select the default team (which should have the same name as your team project). Click Save.

    ![Select the Team to Connect to](images/e2e-eclipse/select-team.png "Select the Team to Connect to")

    > **Note**: Your team name may be different

Configure Branch Policies
-------------------------
In this task you will enforce quality on the master branch by creating branch policies.

1. In Chrome, connect to your VSTS Team Project. Click on Code to open the Code Hub.
1. Click the Repo dropdown and select "Manage Repositories".

    ![Manage Repositories](images/e2e-eclipse/manage-repos.png "Manage Repositories")

1. In the tree, expand the MyShuttle2 repo and click on the master branch. Click the Branch Policies tab.

    ![Open branch policies](images/e2e-eclipse/branch-policies.png "Open branch policies")

1. Check the Protect this branch checkbox.
1. Check "Check for linked work items" and set the radio to required.
1. Under Build validation, click Add build policy. Select MyShuttle2 from the list of build definitions and click Save.

    ![Policy configuration](images/e2e-eclipse/policy.png "Policy configuration")

    > **Note**: You can enforce other policy options like comment resolution and minumum number of reviewers, as well as specify the merge options (like squashing). You can also add default reviewers.

Log a Bug using the Exploratory Test Extension
----------------------------------------------
In this task you will start a test session, discover a bug in the MyShuttle app and log it to VSTS.

1. In the Test extension toolbar of the Exploratory Test extension, click the Play icon to start a testing session.

    ![Start a test sessions](images/e2e-eclipse/start-test-session.png "Start a test sessions")

    > **Note**: The test extension is now recording all of your interactions. You can see the test icon beaker has a green dot indicating that a session is currently running.

1. Enter `http://localhost:8081/myshuttledev` in the toolbar to navigate to the application. Enter `fred` for the username and `fredpassword` for the password and click Log In.

    ![Log in to the app](images/e2e-eclipse/login.png "Log in to the app")

1. On the Dashboard page, click "Access Your Fare History" to navigate to the fare history page.
1. If you look at the totals for the Fare and Driver column in the table, you will note that the total for the driver column is incorrect.
1. Click the Test Extension beaker icon and click the Camera icon (capture image).

    ![Click Add Screenshot](images/e2e-eclipse/click-camera.png "Click Add Screenshot")

1. Capture the grid with the incorrect total. Annotate the image appropriately and click the tick (accept) icon.

    ![Capture an image of the Bug](images/e2e-eclipse/add-screenshot.png "Capture an image of the Bug")

1. Click the Test Extension beaker icon and click flyout (lower right) of the icon with the page and exclamation mark (new bug). From the menu click Create bug.

    ![Create a new bug](images/e2e-eclipse/new-bug.png "Create a new bug")

1. In the title box, enter "Driver total incorrect" and click Save.

    ![Log the Bug](images/e2e-eclipse/log-bug.png "Log the Bug")

    > **Note**: All the pages visited, notes, screenshots and other information from the test session is included as details for the Bug, so you don't have to add these details manually. You also should see a button next to the title box reading "0 Similar". VSTS checks to see if there are bugs already logged with similar titles, therefore minimizing duplicate bugs being logged.

1. Once the bug has been created, click the Stop button in the Test Extension toolbar to end the test session.
1. Navigate to your VSTS team project. Click Work to navigate to the Work Hub. In the toolbar, enter "driver" into the Search Work Items box and press enter or click the magnifying glass icon.

    ![Search for the Bug](images/e2e-eclipse/search-bug.png "Search for the Bug")

1. You should see the Bug that you logged. Take a moment to look at the Repro Steps.

    ![Bug details](images/e2e-eclipse/bug-details.png "Bug details")

1. Assign the Bug to yourself and change the state to Active. Click Save.

    ![Assign the Bug](images/e2e-eclipse/assign-bug.png "Assign the Bug")

Fix the Bug
-----------
In this task you will create a branch of the code to fix the Bug. You will then checkout the branch, fix the bug and commit the code. You will then create a Pull Request to merge the fix into master and see that this triggers the CI/CD pipeline to automatically deploy the fix to the dev environment.

>Note: Use the personal access token (PAT) generated from the "Set up a Docker Build" lab that should be located at: `home/vmadmin/pat.txt`. Otherwise, follow the instructions from that lab again to generate a new PAT. 

1. Open Eclipse if it is not already open. Open the MyShuttle2 project.

1. In Team Explorer change the drop down to "Work Items".  If the dropdown does not show work items connect to your VSTS account via the Team Explorer Home page.

1. If there are no queries saved in VSTS, a query can be created in Eclipse (but not saved at this time). Right-click on the My Queries folder and select "New Query."

    ![New query](images/e2e-eclipse/newquery.png "New query")

1. Run an existing query by double clicking it to find the bug. Or, right click in the New Query panel and select "Run Query." The output of the query will show the bug. Note the ID value of the bug.

    ![Confirm the bug is correctly assigned and in VSTS](images/e2e-eclipse/findbug.png "Confirm the bug is correctly assigned and in VSTS")

    > **Note**: If you do not see the bug, ensure that it is assigned to you, since by default only work items assigned to you will appear in the work item list.

1. Create a new branch

    ![New branch](images/e2e-eclipse/createbranch.png "New branch")

1. In the dialog, change the branch name to "totalsBug" and click Create.

    ![New branch](images/e2e-eclipse/createbranch2.png "New branch")


1. In the project view of Eclipse, browse to `src/main/java/com.microsoft.example.servlet` and open the LoginServlet class.

1. Around line 35, you will see what is causing the bug: the `totalDriverFee` is being calculated but the `driverFeeTotal` session attribute is being set to `totalFareForDriver` (this looks like a classic copy/paste error).

    Change this line of code:
    ```java
        session.setAttribute("driverFeeTotal", totalFareforDriver);
    ```
    to 
    ```java
        session.setAttribute("driverFeeTotal", totalDriverFee);
    ```

1. Commit your changes by right clicking the file and selecting Team->Commit. Enter "Fixing totals bug #{ID of bug}" as the commit message. By putting the # symbol followed by an ID of a work item in a commit message, VSTS will automatically associate the work item with the commit when it's pushed to VSTS. In the example of the screenshot, the ID is #698. Click "Commit and Push" to push the changes to VSTS.

    ![Commit and Push](images/e2e-eclipse/eclipse-newcommit.png "Commit and Push")

1. If a window pops up that prompts for credentials, use the following values: 

    | Name | Value |
    |---|---|
    | User | `_VSTS_Code_Access_Token` |
    | Password | `{PAT that you copied earlier}` |
        
    ![Login to Eclipse](images/packagemanagement/eclipse-login.png "Login to Eclipse")

    In the Push commits dialog click the Push button.

1. Now that the fix has been pushed to VSTS on a branch, you can create a Pull Request. This will be done in VSTS following the standard process for pull requests. Under the Code hub, click on Files in the MyShuttle2 repo and there should be a notification that you updated the `totalsBug` branch. Click the link next to it, "Create a pull request." 

    ![Create Pull Request](images/e2e-eclipse/pullrequest.png "Create Pull Request")

1. Then, in the pull request panel, click "Create" to create the pull request. Note that the bug is associated with the commit. 

    ![Create Pull Request](images/e2e-eclipse/pullrequest2.png "Create Pull Request")

1. Once the PR has been created, right-click it in the PR list and click Open in Browser. You should see that the build is running (this is the build mandated by the Branch Policy you set up earlier).

    ![Build is running to validate the PR](images/e2e-eclipse/pr-overview.png "Build is running to validate the PR")

    > **Note**: If there was a merge conflict, VSTS would warn you on the overview page. If there is no warning to this effect, then Git will be able to auto-merge the PR into the target branch.

    > **Note**: You configured the release to only trigger when successful builds off the master branch are available. Since this build is not building from the master branch, these changes will not yet be deployed.

1. Click on the Files tab to open the file compare. Note the changes.

    ![PR File Compare](images/e2e-eclipse/PR-file-compare.png "PR File Compare")
    
    > **Note**: You can comment on code or files in the PR and have conversations with the team throughout the review process.

1. Click Approve to approve the PR.
1. Now that the policies have been fulfilled, you can complete the PR which will merge the changes into master (the target branch). Click Complete to do the merge.
1. In the dialog, accept the defaults and click Complete merge.

    ![Complete the merge](images/e2e-eclipse/complete-merge.png "Complete the merge")

1. The PR completion triggers a new build off the master branch, which in turn will trigger a release. _It also transitions the Bug work item to Resolved_.
1. Click on Builds to watch your build. When the build completes, you will see the unit test and code coverage results as well as SonarQube analysis and quality gates (if you have configured SonarQube integration).
1. Click on Releases and open the latest release which should have triggered off the PR merge build completion event.
1. On the Release Summary page, you will see the linked Bug work item.

    ![Linked work item in Release](images/e2e-eclipse/linked-bug-release.png "Linked work item in Release")

1. Click on commits to see the incoming commits for this release. There is the commit to fix the bug as well as the commit to merge into master.

    ![Linked commits](images/e2e-eclipse/linked-commits.png "Linked commits")

1. Click on the Tests tab to see the test results. The UI tests should be passing.
1. Open the MyShuttle2 app by navigating to `http://localhost:8081/myshuttledev`. Log in again and verify that the totals column is correct and the Bug has been fixed.

    ![The bug has been fixed](images/e2e-eclipse/bug-fixed.png "The bug has been fixed")