# IBM Client Developer Advocacy App Modernization Series

## Lab - Automated updates of containerized applications from SCM commits

### Creating a CI/CD Pipeline for deployment to IBM Cloud Private using Jenkins

## Overview

In this lab you will  be connecting your Git repository with the Plants by WebSphere app to a Continuous Integration/Continuous Deployment pipeline built with Jenkins that will deploy to a IBM Cloud Private  cluster.

You will see a number of environment variables prefaced with a `$` used in the lab commands. If you have already set these variables, you may copy and paste the commands as is. If you have not, replace the environment variables with values from your lab instructor.

## Setup

If you haven't already:

1.  Complete *Working with Helm*  by following the instructions [here](https://github.com/IBMAppModernization/app-modernization-helm-lab-icp)

2.  If following this lab as part of an IBM instructor led workshop, login into your terminal with the credentials provided to you and complete all environment setup steps. Check your environment with these commands (all should have output):

    ```text
    echo $CLUSTER_URL
    echo $USERNAME
    ```

3.  Login to your designated ICP instance from the command line

    ```text
    cloudctl login -a $CLUSTER_URL
    ```

### Step 1: Set up the CI/CD pipeline

In this section we will be connecting our cloned Git repo of [this app](https://github.com/IBMAppModernization/app-modernization-plants-by-websphere-jee6)  to set up a Continuous Integration/Continuous Deployment pipeline built with Jenkins. This pipeline contains 4main  steps as follows:

  | Stage                         | Purpose                                                                        |
  | ----------------------------- | ------------------------------------------------------------------------------ |
  | Build Application ear File    | Pulls in dependencies from Maven and packages application into .ear file       |
  | Build Docker Image            | Builds the Docker image based on the Dockerfile                                |
  | Push Docker Image to Registry | Uploads the Docker image to the Docker image registry within ICP               |
  | Deploy New Docker Image       | Updates the image tag in the Kubernetes deployment triggering a rolling update |

More details of this pipeline can be found in the [Jenkinsfile](https://raw.githubusercontent.com/IBMAppModernization/app-modernization-plants-by-websphere-jee6/master/Jenkinsfile.ext-icp).

1.  Log into Jenkins using the URL provided to you by your instructor with the credentials provided to you

2.  The pipeline should have already been created for you.

    ![Jenkins initial page](images/ss1.png)

3.  Click on your pipeline to open it and then click on the **Configure** link in the navigation area at the left to change it's properties

4.  Scroll down to the **Build Trigger** section and select **GitHub hook trigger for GIT SCM polling**

    ![Build Triggers](images/ss2.png)

5.  Scroll down to the **Pipeline** section and find the **Definition** drop down menu. Select **Pipeline script from SCM** and for **SCM** select **Git**.

6.  For **Repository URL** enter the url to the cloned repository that you forked earlier (i.e. `https://github.com/[your username]/app-modernization-plants-by-websphere-jee6.git`)

7.  Verify that the  **Script Path** is set to `Jenkinsfile.ext-icp`

    ![pipeline config](images/ss3-ext.png)

8. Click **Save**.

### Step 3: Manually trigger a build to test pipeline

1.  In Jenkins in the navigation area on the left click on **Build with Parameters**. Accept the defaults of the parameters and click on **Build**

2.  To see the console output click on the build number in the **Build History** and then click on **Console Output**

    ![Console output](images/ss4.png)

3.  If the build is successful the end of the console output should look like the following:

    ![End of console output](images/ss5.png)

    The Stage View of the pipeline should look like the following:

    ![Stage view](images/stages.png)

### Step 4: Trigger a build via a commit to Github

Now you'll configure Github to trigger your pipeline whenever code is committed.

1.  Go back to Github and find your cloned repository

2.  Click on the repository settings

    ![Settings](images/ss6.png)

3.  Under **Options** select **Webhooks** and click **Add webhook**

    ![Add webhook](images/ss7.png)

4.  For the Payload URL use `<Jenkins URL>/github-webhook/`  where `<Jenkins URL>` is the  URL you used  to login to Jenkins (**Note** Don't forget the trailing `/`)

5.  Change content type to **application/json**

6.  Accept the other defaults and click **Add webhook**

    ![Add webhook](images/ss8.png)

7.  In the Github file browser drill down to *pbw-web/src/main/webapp/promo.xhtml*

8.  Click on the pencil icon to edit **promo.xhtml**  and on line 95 locate the price of the Bonsai Tree

9.  Change  `$30.00 each` to `<strike>$30.00</strike> $25.00 each`

    This will show the price of the Bonsai Tree as being reduced even more

    ![Reduce Bonsai price](images/ss10.png)

10. At the bottom of the UI window add a commit message and click on **Commit changes**

11. Switch back to Jenkins  and open the pipeline that you were working on  earlier.

12. Verify that your pipeline  starts building.

13. When the pipeline is finish deploying, launch the app to verify the change you made.

14. Launch the IBM Cloud Private Web UI using the ICP Master URL given to you by your instructor and login in.

15. In the Navigation area on the left expand **Workloads** and select **Helm Releases**

16. Look for your Helm Release in the list and click on the **Launch** link on the right

17. Verify that the app's UI opens in another tab and that the proce of the bonzai tree has changed.

    ![Price reduced](images/ss9.png)

## Cleanup

Free up resources for subsequent labs by deleting the Plants by Websphere app.

1. Run the following command to delete the app from your terminal session:

   ```bash
   helm delete --purge pbw-liberty-mariadb-$USERNAME --tls
   ```


## Summary

You created a Jenkins pipeline to automatically build and deploy an app that has been updated in Github .
