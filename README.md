Deploy Springboot Microservices App into Amazon EKS Cluster using Jenkins Pipeline and Kubectl CLI Plug-in | Containerize Springboot App and Deploy into EKS Cluster using Jenkins Pipeline
https://www.coachdevops.com/2022/01/deploy-springboot-microservices-app_11.html


Setup EKS Cluster using eksctl and Deploy Springboot Microservices into EKS using Jenkins Pipeline
https://www.youtube.com/watch?v=C1BlN66s9yo

github.com/akannan1087 - github

Home: https://www.coachdevops.com/2022/01/deploy-springboot-microservices-app_11.html

A-https://www.coachdevops.com/2020/04/install-jenkins-ubuntu-1804-setup.html
1.Change Host Name to Jenkins: sudo hostnamectl set-hostname Jenkins
2.Perform update first: sudo apt update
3.Maven Installation: sudo apt install maven -y (PS: Java also install with maven)
4.Verification: mvn -version and java -version
5.Jenkins Setup: check at above A-url
	a)Add Repository key to the system
	b)Append debian package repo address to the system
	c)Update Ubuntu package: sudo apt update
	d)Install Jenkins: sudo apt install jenkins -y
6.Access Jenkins in web browser, Install suggested packages and create user

B-https://www.coachdevops.com/2022/02/create-amazon-eks-cluster-by-eksctl-how.html
7.Install AWS CLI: https://www.coachdevops.com/2020/10/install-aws-cli-version-2-on-linux-how.html
8.Install eksctl: https://www.coachdevops.com/2020/10/install-eksctl-on-linux-instance-how-to.html
9.Install kubectl: https://www.coachdevops.com/2022/05/install-kubectl-on-ubuntu-instance-how.html 

10.Create IAM Role with Administrator Access: (eg: Name: ekscluster-admin-role) https://www.coachdevops.com/2022/02/create-amazon-eks-cluster-by-eksctl-how.html
11.Assign the role to EC2 instance
12.Create EKS Cluster with two worker nodes using eksctl with B-url (PS:Switch Jenkins user (sudo su - jenkins))
   
13.EKS will take around 20 min. mean while install docker on EC2 instance
C-https://www.coachdevops.com/2020/05/docker-jenkins-integration-building.html
14.Docker Jenkins Integration: follow C-url
15.Docker, Docker pipeline and Kubernetes CLI plug-ins are installed in Jenkins at C-url

16.
	Step #1 - Create Maven3 variable under Global tool configuration in Jenkins
	Step #2 - Create Credentials for connecting to Kubernetes Cluster using kubeconfig

====================================================================================================================================================================
**Manage Jenkins**
Tools: Maven Installation
Plugins: Docker, Docker Pipeline,Kubernetes CLI Plugin
Credentials: "Secrect file " for Kubernetes 
====================================================================================================================================================================
Pipeline Script:

pipeline {
    agent any

    tools {
        maven "Maven3"
    }
    
    environment {
        registry = '992382772822.dkr.ecr.ap-south-1.amazonaws.com/my-docker-repo'
    }
    stages {
        stage('**1.Checkout**') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/SriramaInboxYahoo/springboot-app']])
            }
        }
    
        stage('**2.Build Jar**'){
            steps {
                sh "mvn clean install"
            }
        }    
        
        stage('**3.Build Docker Image**') {
            steps {
                script {
                    docker.build registry
                }
            }
        }
        
        stage('**4.Push Docker Image into ECR**') {
            steps {
                sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 992382772822.dkr.ecr.ap-south-1.amazonaws.com"
                sh "docker push 992382772822.dkr.ecr.ap-south-1.amazonaws.com/my-docker-repo:latest"
            }
        }
        
        stage('**5.K8S Deploy**') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                    sh "kubectl apply -f eks-deploy-k8s.yaml"
                    
                }
            }
        }
    }    
}
====================================================================================================================================================================
	Step #3 - Create a pipeline in Jenkins
