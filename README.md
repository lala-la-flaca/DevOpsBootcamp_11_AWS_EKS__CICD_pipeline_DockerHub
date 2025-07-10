# AWS EKS
## 📦 Demo 6
This project is part of **Module 11: Kubernetes on AWS (EKS)** in the **TWN DevOps Bootcamp**.It shows how to set up a **complete CI/CD pipeline** using Jenkins to deploy an application to an Amazon EKS cluster, while pulling container images from a **private Docker Hub registry**.

[GitLab Repo](https://gitlab.com/devopsbootcamp4095512/devopsbootcamp_8_jenkins_pipeline/-/tree/complete_pipeline_EKS_DockerHub?ref_type=heads)

## 📌 Objective
Build a CI/CD pipeline that:
- Builds a Docker image of the app
- Pushes the image to a **private Docker Hub registry**
- Pulls the image into an **EKS deployment**
- Deploys the app to the EKS cluster automatically using Jenkins

## 🚀 Technologies Used

- **kubectl**: CLI to interact with Kubernetes.
- **eksctl**: CLI tool for creating and managing EKS clusters
- **DigitalOcean**: hosting Jenkins server.
- **AWS EKS**: AWS kubenertes cluster
- **Docker Hub**: Private Docker registry
- **Java/Maven**: Java/Maven app from the previous demo when using the increment version
- **Docker**: Containarization.
   
## 📋 Prerequisites
- Ensure you have an AWS Account
- Kubectl is installed and configured to connect to the Kubernetes cluster.
- Jenkins server is running
- We are going to use the Java-Maven-App repository from the previous demo.
  
## 🎯 Features
- Create k8 manifest files for Deployment and Service configuration.
- Update the deploy step in the CI/CD to deploy the newly built application image from DockerHub to EKS cluster.
- Complete CI/CD pipeline:
  - CI step: Increment version
  - CI step: Build artifact for Java/Maven app
  - CI step: Build and push to DockerHub
  - CD step: Deploy new application to EKS
  - CD Step: Commit version update.
       
## 🏗 Project Architecture



## ⚙️ Project Configuration



### Creating Deployment and Service YAML files
1. Create a New branch to your Java-Maven-App repository, which forks from the Java Increment version demo.
2. Create a new Kubernetes directory.
3. Create the Deployment and Service file.
4. Open the deployment file and secret file, and copy the baseline file.
5. Open the deployment file and set a dynamic Image name through an ENV Variable ($IMAGE_NAME), and replace the java-maven-app with an ENV variable ($APP_NAME)
   ```bash
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: $APP_NAME
        labels:
          app: $APP_NAME
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: $APP_NAME
        template:
          metadata:
            labels:
              app: $APP_NAME
          spec:
            imagePullSecrets:
              - name: aws-registry-key
            containers:
              - name: $APP_NAME
                image: lala011/demo-app:$IMAGE_NAME
                imagePullPolicy: Always
                ports:
                  - containerPort: 8080
   ```
6. Open the service file and replace the java-maven-app with an ENV variable ($APP_NAME)
   ```bash
       apiVersion: v1
       kind: Service
       metadata:
         name: $APP_NAME
       spec:
         selector:
           app: $APP_NAME
         ports:
           - protocol: TCP
             port: 80
             targetPort: 8080
   ```
7. Update the Jenkins file with the environment block to add the APP_NAME env variable and the AWS plugin env variables
   ```bash
      environment {
          KUBECONFIG = "${env.WORKSPACE}/kubeconfig"
          AWS_REGION = 'us-east-2'
          CLUSTER_NAME = 'demo-cluster'
          APP_NAME = 'java-maven-app'
  
      }
   ```
8. Update Jenkinsfile on the deploy block to  apply the deployment and service yaml files. To pass the deployment and service yaml files to the Jenkinsfile, we are going to use the envsubst command line.

    <details><summary><strong> envsubst </strong></summary>
    envsubst is a command-line utility that performs variable substitution on text by replacing environment variables within a string or file with their corresponding values. It's commonly used to dynamically generate configuration files or other text-based artifacts by incorporating environment variables into templates. Here we are passing the deployment and service yaml file to envsubs
    </details>
```bash
    stage("deploy") {
    
                 steps {
    
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'AWS_jenkins_key'
                    ]]){
                        script {
                            //gv.deployApp()
                            echo "Deploying app"
                            echo 'Generating kubeconfig...'
                            sh '''
                                aws eks update-kubeconfig \
                                  --region $AWS_REGION \
                                  --name $CLUSTER_NAME \
                                  --kubeconfig $KUBECONFIG
                            '''
                            echo 'deploying docker image...'
                            sh 'kubectl get nodes'
    
                            sh '''
                                pwd
                                ls
                                cd java-maven-app/
                                envsubst < kubernetes/deployment.yaml | kubectl apply -f -
                                envsubst < kubernetes/service.yaml | kubectl apply -f -
                            '''
                        }
                    }
                }
            }
```

### Installing Gettext-base on Jenkins
1. SSH to the droplet to access the Jenkins server
2. Access to Jenkins container as root user.
3. Install gettext-base
   ```bash
     apt-get update
     apt-get install gettext-base
   ```
4. Exit the container

### Creating a Secret for DockerHub
For this Demo, the secret file is a file that is only created once; therefore, we do not want to include it in the pipeline. We are going to create the Secret from the host. Each NameSpace has its  Secret file
1. Ensure your EKS cluster is running.
   ```bash
   kubectl get node
   ```
2. Create Secret
   ```bash
     kubectl create secret docker-registry my-registry-key \
     --docker-server=docker.io \
     --docker-username=xxxx \
     --docker-password=xxxxx
   ```
3. Verify that the secret was created
   ```bash
     kubectl get secret
   ```
4. Add the secret to the deployment.yaml file
   ```bash
     spec:
        imagePullSecrets:
          - name: my-registry-key
  ```
5. Commit changes
   
