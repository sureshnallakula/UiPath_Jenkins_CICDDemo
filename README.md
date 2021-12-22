# UiPath_Jenkins_CICDDemo
How to setup a basic UiPath CICD pipeline with Jenkins. This pipeline tracks a Github repository for changes and as soon as a new version of code is committed, it pushes a new build and to the UiPath orchestrator in an automated fashion.

Note: This tutorial is intended as a learning concept to get a pipeline up and running to explore CICD further. It is not intended as an approach for deploying code to production.

You can preview this implementation in the following video: UiPath, Jenkins CICD Pipeline

What you’ll need:

- Admin rights to an orchestrator instance (UiPath Community or Enterprise Edition)
- Jenkins installed on your Windows PC
- A Github account and UiPath integrated with it

High level steps:

1. Install Jenkins and UiPath Plugin
2. Clone Github Repo
3. Setup Jenkins Pipeline
4. Connect Git Repository

In each stage I mention the pitfalls that you have to watch out for.

I have also included additional references at the end of this post. A special thanks to Satish Prasad - his post on RPABotsWorld.com 30 helped me to set this up. I have linked his article in the references section as well.

1. Install Jenkins and UiPath Plugin

- Install Java Development Kit (JDK)
- Add JDK to PATH
- Download and install Jenkins
- Run Jenkins on localhost 8080
- Install UiPath Plugin within Jenkins

- Take care to install a version of JDK that Jenkins is compatible with. As on this date, this version is 11. Find out the latest supported version here: Java requirements 13
- During the installation, select Run service as LocalSystem even though it isn’t recommended

Note: If your Jenkins workspace is set to a folder within the WINDOWS folder (which is usually the case… a path such as C:\WINDOWS\system32\config\systemprofile\AppData\Local\Jenkins.jenkins\workspace), you need to carry out an additional step for the UiPath plugin to work. You need to change this folder to one outside WINDOWS. To do this, navigate to the ‘config.xml’ file within your Jenkins folder to modify the workspace and the builds directories…

<workspaceDir>C:/JenkinsRoot/${ITEM_FULL_NAME}/workspace</workspaceDir>
<buildsDir>C:/JenkinsRoot/${ITEM_FULL_NAME}/builds</buildsDir> 
… and restart Jenkins.

What this achieves is that it changes my workspace and build directories to ‘C:/JenkinsRoot’. Feel free to come up with a more creative name.

Reference: How to change workspace and build record Root Directory on Jenkins?

2. Clone Github Repo

- Clone the Github repo UiPath_Jenkins_CICDDemo from the following link: GitHub - descriptify/UiPath_Jenkins_CICDDemo 106
- Open the Jenkinsfile in the repo with your favourite IDE within the repository and modify it:

Note: The Jenkinsfile is written in Groovy syntax. Within it, you can see the various stages of the pipeline:
Build → Test → Deploy to UAT-> Deploy to Production
I’ve included the full code within the Jenkinsfile at the end of this post.

To replicate the demo, you need to input your own orchestrator details into this file against the following variables:

UIPATH_ORCH_URL = "https://cloud.uipath.com/"
UIPATH_ORCH_LOGICAL_NAME = "anupaminc"
UIPATH_ORCH_TENANT_NAME = "Descriptify"
UIPATH_ORCH_FOLDER_NAME = "Default"

You can find these values in your automation cloud instance. Navigate to Admin → Tenant → API Access

3. Setup Jenkins Pipeline

Before we get to the pipeline itself, we need to create a credential for the API User Key we obtained earlier to the UiPath Orchestrator.

In Jenkins, add a global credential for the API key. Navigate to Jenkins → Manage Jenkins → Manage Credentials. You should see the following screen:

Click on ‘global’ → Add Credentials. In the ‘Kind’ dropdown, select Secret Text.

Enter your API User Key in the secret field. In the ID field, be sure to enter “APIUserKey” as the value. This is how the code in the Jenkinsfile refers to this credential. Click ‘OK’ after including a helpful description.

We are now ready to create our Jenkins pipeline. Navigate to Jenkins → New Item. Enter a suitable name, select Multibranch Pipeline and click OK

In the Branch Sources tab, select Add Source → Github:

In the Repository HTTPS URL field, enter the URL to your Github repository and click Save.

That is pretty much it… If everything is setup perfectly, Jenkins should show you the following screen with the status as “Finished: SUCCESS.”


4. Connect Git Repo for automated builds when new commits are made

As with most things, there is an easy way and a hard, but elegant way to do this:

The easy way is to check for new builds is to have Jenkins periodically poll your github repository and check if new code has been committed. You can do this by navigating to Jenkins Pipeline → Configure → Scan Repository Triggers.

The limitation here is that updates happen only at a set frequency - not as soon as code is committed. Another factor is that Jenkins imposes limitations on polling Git repositories for updates.

The hard, but elegant way is to configure webhooks:

Github Repo → Settings → Webhooks → Add Webhook

On taking that path, you should see the following form. Here, you need to specify the URL to your Jenkins server followed by “/github-webhook/”

Select “application/x-www-form-urlencoded” as the type and the “Just the push event” radio button before creating the webhook.



