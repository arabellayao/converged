# Integrate Jenkins with GitHub and Oracle Cloud Infrastructure

## Introduction

This lab will demonstrate how to integrate Jenkins with GitHub and Oracle Cloud Infrastructure Services and build a pipeline.

GitHub provides webhook integration, so Jenkins starts running automated builds and tests after each code check-in. A sample web application Grabdish is modified and re-deployed as part of the CI/CD pipeline, which end users can access from the Container Engine for the Kubernetes cluster.

Estimated Time: 15 minutes

### Objectives

* Execute GitHub Configuration
* Execute Jenkins Configuration
* Configure a Pipeline

### Prerequisites

* This lab presumes you have already completed the earlier labs.
* As this is a demonstration of Jenkins/GitHub integration for CI/CD, **you must use your own GitHub account to run it.** We assume you completed this step in Setup lab.

## Task 1: Configure Jenkins Pipeline

1. A service account is needed to allow Jenkins to update the grabdish Kubernetes cluster. To create a service account, connect to the cloud shell and execute the following command.

     ```bash
     <copy>
     kubectl apply -f $DCMS_CICD_SETUP_DIR/kubernetes/service-account.yaml
     </copy>
     ```

     Kubernetes will create a secret token bound to the service account. Using below command retrieve the secret:

     ```bash
     <copy>
     $(kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep kube-cicd | awk '{print $1}') -o jsonpath='{.data.token}' | base64 -d)
     </copy>
     ```

     Copy the secret token - you will use it in the next steps when creating a secret credential.

2. Open a new browser tab and login into your Jenkins console as ADMIN user using Jenkins Public IP from **Lab 2: Task 1** and the password you supplied during Jenkins setup run: `https://jenkins.example.com`

   ![Jenkins Login](images/jenkins-login.png " ")

3. Navigate to the Jenkins credentials store to create credentials

   1. From the Home page, click on `Manage Jenkins`.

   ![Jenkins Credentials](images/jenkins-creds.png " ")

   2. From the Manage Jenkins page, Under Security, click `Manage Credentials`.

   3. Hover over (`global`), the domain for the Jenkins Store (under Stores scoped to Jenkins).

   4. Click on the dropdown.

   5. Click on `Add Credentials` and add the secret text credentials.

   ![Jenkins Credentials](images/jenkins-store.png " ")

   ![Jenkins Secret](images/jenkins-secret-creds.png " ")

     Kind: `Secret text`
     Scope: `Global`
     Secret: < Paste content of service account secret token created above >
     Click `Create`

    Add another credential by clicking **Add Credentials** in the left hand navigation bar.

     Kind: `Username with password`
     Username: <tenancy_namespace>/oracleidentitycloudservice/username>
     Password: <Paste auth token as password created during infra runtime - you can retrieve the docker auth token through logs >
     Click `Create`

     > **Note:** Note the "Username with password" credential's ID for the next steps.

## Task 2: Configure Maven Tool

1. Navigate to `Manage Jenkins` and then click on `Global Tools Configuration`

     ![Jenkins Tool Configuration](images/jenkins-tool-config.png " ")

     Under `Maven > Maven Installation`, add Maven with name **maven3**
     Click `Save`

## Task 3: Create a New Pipeline

1. On Jenkins Dashboard, click on `New Item` and enter the name for the item: `Demo`.

2. Select `Pipeline` and click `OK`.

     ![Jenkins Pipeline ](images/jenkins-pipeline.png " ")

3. Under `Build Triggers`, select `GitHub hook trigger for GITScm polling`.

4. Copy and Paste Jenkinsfile from the repository workshops/dcms-cicd/jenkins/Jenkinsfile

     ![Jenkinsfile](images/jenkins-pipeline-file.png " ")

     Under `environment` section of Jenkinsfile, supply the missing values:

        ocir_credentials_id = ""
        region = ""
        namespace = ""

## Task 4: Add GitHub WebHook

1. Log into GitHub and click on the repository which has been integrated with Jenkins. Navigate to Settings -> Webhooks.

     ![New Item](images/github-webhooks.png " ")

2. On GitHub settings - add a WebHook with the IP address of Jenkins console: `http://jenkins.example.com/github-webhook/`

> **Note:** Replace the Jenkins example with Jenkins public IP address. **The trailing slash is important**

You may now **proceed to the next lab.**.

## Acknowledgements

* **Authors** - Irina Granat, Consulting Member of Technical Staff, Oracle MAA and Exadata; Norman Aberin, Member of Technical Staff
* **Last Updated By/Date** - Irina Granat, June 2022
