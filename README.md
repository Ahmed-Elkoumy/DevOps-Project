# DevOps-Project


**SUMMARY**

**I've been developing to showcase my skills as a junior DevOps engineer. The project involves deploying a Node.js application with a Postgres database on an AWS EKS cluster. I also set up an ECR repository, EBS volumes, an Ingress controller, and monitoring using Prometheus and Grafana. To make things easier, I created a script called build.sh that automates the deployment process.**

**Before running the script, there are a few things you need to customize according to your own AWS environment and deployment requirements. Open the build.sh script in a text editor and update the variables at the top of the script. You'll need to specify your EKS cluster name, AWS region, AWS account ID, repository name, domain, and a few other configurations. Make sure you save the changes after updating the variables.**

**Once you've updated the variables, you can run the build.sh script. It will take care of setting up the infrastructure on AWS, building a Docker image for your Node.js app, and deploying it to the Kubernetes cluster. The script will also generate a random password for the Postgres database and create a secret key for it.**



==============================================================================================================================================================================



**This project consists of scripts and Kubernetes manifests for deploying the Nodejs application with Postgres Database on an AWS EKS cluster with an accompanying ECR repository and EBS volumes. The deployment includes setting up an Ingress controller, monitoring with Prometheus and Grafana, and a continuous deployment pipeline.**
Before you run the script
Update Script Variables:

The build.sh script contains a set of variables that you need to customize according to your AWS environment and deployment requirements. Here's how you can update them:

    Open the build.sh script in a text editor of your choice.

    Update the variables at the top of the script with your specific configurations.

   


After updating these variables, the script will use them to deploy your application, ensuring that the correct resources are targeted and that your application is accessible via your specified domain.



   
    grafana:
      adminUser: admin
      adminPassword: admin
      enabled: true
      ingress:
        enabled: true
        ingressClassName: nginx
        annotations:
          cert-manager.io/cluster-issuer: letsencrypt-production
        hosts:
          - grafana.[YOUR_DOMAIN]
        tls:
          - secretName: grafana-tls
            hosts:
              - grafana.[YOUR_DOMAIN]



The build.sh script automates the process of setting up the infrastructure on AWS, building a Docker image for the Go Survey app, and deploying it to the Kubernetes cluster. 


How to Run

Clone the repository and navigate to the root directory.

Make the deployment script executable:

chmod +x build.sh

Run the deployment script:

./build.sh

Follow the post-deployment steps printed by the script to update your DNS records with the provided Ingress URL.CI/CD Workflows

This project is equipped with GitHub Actions workflows to automate the Continuous Integration (CI) and Continuous Deployment (CD) processes.
Continuous Integration Workflow

The CI workflow is triggered on pushes to the main branch.

=

Continuous Deployment Workflow

The CD workflow is triggered upon the successful completion of the CI workflow.
GitHub Actions Secrets

The following secrets need to be set in your GitHub repository for the workflows to function correctly:

    AWS_ACCESS_KEY_ID: Your AWS Access Key ID.
    AWS_SECRET_ACCESS_KEY: Your AWS Secret Access Key.
    KUBECONFIG_SECRET: Your Kubernetes config file encoded in base64.

Setting Up GitHub Secrets for AWS

Before using the GitHub Actions workflows, you need to set up the AWS credentials as secrets in your GitHub repository. The included github_secrets.sh script automates the process of adding your AWS credentials to GitHub Secrets, which are then used by the workflows. To use this script:

    Ensure you have the GitHub CLI (gh) installed and authenticated.

    Run the script with the following command:

    ./github_secrets.sh

This script will:

    Extract your AWS Access Key ID and Secret Access Key from your local AWS configuration.
    Use the GitHub CLI to set these as secrets in your GitHub repository.

Note: It's crucial to handle AWS credentials securely. The provided script is for demonstration purposes, and in a production environment, you should use a secure method to inject these credentials into your CI/CD pipeline.

These secrets are consumed by the GitHub Actions workflows to access your AWS resources and manage your Kubernetes cluster.
Adding KUBECONFIG to GitHub Secrets

For the Continuous Deployment workflow to function properly, it requires access to your Kubernetes cluster. This access is granted through the KUBECONFIG file. You need to add this file manually to your GitHub repository's secrets to ensure secure and proper deployment.

To add your KUBECONFIG to GitHub Secrets, follow these steps:

    Encode your KUBECONFIG file to a base64 string:

    cat ~/.kube/config | base64

    Copy the encoded output to your clipboard.

    Navigate to your GitHub repository on the web.

    Go to Settings > Secrets > New repository secret.

    Name the secret KUBECONFIG_SECRET.

    Paste the base64-encoded KUBECONFIG data into the secret's value field.

    Click Add secret to save the new secret.

This KUBECONFIG_SECRET is then used by the CD workflow to authenticate with your Kubernetes cluster and apply the required configurations.

Important: Be cautious with your KUBECONFIG data as it provides administrative access to your Kubernetes cluster. Only store it in secure locations, and never expose it in logs or to unauthorized users.
Interacting with the application

To access and manage the Database from your local machine, you can use k9s to port forward the service and then connect to it using BeeKeeper.
Accessing the Service with k9s

    Open k9s in your terminal.
    Navigate to the services section by typing :svc and pressing Enter.
    Search for the service named postgres-service.
    With the postgres-service highlighted, press Shift+F to set up port forwarding to your local machine.

Connecting to the Database

Once you've port forwarded the postgres-service:

    Open Beekeeper.

    Connect to the Postgres Database using the localhost address and the port 5432.

    Use the USERNAME you added in the k8s environment variable.

    Get the PASSWORD from k8s secrets using k9s.

        Navigate to secrets

        Find db-password-secret

        Tab on x in your keyboard to decode the generated password

Adding Data to the Database with Beekeeper

Create a table in the database and insert data.

    Run the following query to create table.

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    body TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

    Run the following query to insert data into the table.

INSERT INTO posts (title, body) VALUES ('First Post', 'This is the first post.');
INSERT INTO posts (title, body) VALUES ('Second Post', 'This is the second post.');
INSERT INTO posts (title, body) VALUES ('Third Post', 'This is the third post.');

    Check the application webpage to see the updates.

After running the queries, you should be able to verify that the new entries have been added to the database and showed on the webpage.
Destroying the Infrastructure

In case you need to tear down the infrastructure and services that you have deployed, a script named destroy.sh is provided in the repository. T

Before you run

Open the destroy.sh script.

Ensure that the variables at the top of the script match your AWS and Kubernetes settings:

REGION="REGION"
REPOSITORY_NAME="ECR_REPOSITORY_NAME"

How to Run the Destroy Script

Save the script and make it executable:

chmod +x destroy.sh

Run the script:

./destroy.sh

This script will execute another script ecr-img-delete.sh which will delete all the images on the ECR to make sure the ECR is empty then terraform commands to destroy all resources related to your deployment. It is essential to verify that the script has completed successfully
