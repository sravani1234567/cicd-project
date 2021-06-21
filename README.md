DevOps Exercise #1
Deploy a web application to a cloud provider of your choice. This web application can be something you have written yourself or an open-source project.
Deploy the web application as a Docker Container.
Deploy the Docker Container using Kubernetes.
Any supporting infrastructure should be configured and deployed as code (e.g. Terraform)
Bonus points for any build and deployment automation employed in the deployment of the web application.
Bonus points for demonstrating the ability to deploy, destroy and re-deploy the web application and any supporting infrastructure.
Include all code and artefacts you create to complete this exercise within this repository for review.
Steps to execute

Create AWS Account
Get a Amazon ec2 instance
Install terraform in it curl -O https://releases.hashicorp.com/terraform/0.14.10/terraform_0.14.10_linux_amd64.zip sudo su unzip terraform_0.14.10_linux_amd64.zip -d /usr/bin/ terraform -v
Configure aws
aws configure
provide
Access Key ID:,
Secret Access Key:
region:us-east-2
output: json
Also configure "Access key id" and "Secret Key" according to https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html This is required in terraform.

install git
yum install git -y

git clone https://github.com/vilasvarghese/devops-exercise-1-infra-vilasvarghese
Replace this with your account if you have forked.

cd devops-exercise-1-infra-vilasvarghese/terraform

Make necessary configurational changes to
https://github.com/vilasvarghese/devops-exercise-1-infra-vilasvarghese/blob/main/terraform/variables.tf
https://github.com/vilasvarghese/devops-exercise-1-infra-vilasvarghese/blob/main/terraform/terraform.tfvars

terraform init

terraform plan

terraform apply

Launch an ec2 instance
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
sudo ln -s /usr/local/bin/eksctl /usr/bin/eksctl
eksctl version
eksctl create cluster --name my-cluster --region us-east-2 --fargate

Setup kubectl to work with remote kubernetes master according to the instructions given in the link
https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html
since we are using amazon instance - it already has aws.
Connect to jenkins instance and do the following

N.B: This would allow the jenkins server instance to work with a remote kubernetes (setup kubectl to work remotely in aws)

aws configure
aws sts get-caller-identity
aws update-kubeconfig
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
cp kubectl /usr/local/bin/
sudo ln -s /usr/local/bin/kubectl /usr/bin/kubectl
chmod 777 /usr/local/bin/kubectl

Setup jenkins to execute a ci/cd job Login to :8080 Find the password from sudo cat /var/lib/jenkins/secrets/initialAdminPassword and enter the same.
Install Docker plugin and GitHub Integration Plugin Manage Jenkins → Manage Plugins → Available.

Create Credentials Docker Hub: Click Credentials → global → Add Credentials, choose Username with password as Kind, enter the Docker Hub username and password and use dockerHubCredentials for ID.

GitHub: Click Credentials → Global → Add Credentials , choose Username with password as Kind, enter the GitHub username and password and use gitHubCredentials for ID. (N.B: For now i have hardcoded this in the yaml files - kindly modify the same rather than creating these credentials)

Create a Freestyle job (not pipeline) Copy paste the following script

gitHub repo: https://github.com/vilasvarghese/devops-exercise-1-infra-vilasvarghese Modify the branch from master to main Copy the contents of https://github.com/vilasvarghese/devops-exercise-1-infra-vilasvarghese/blob/main/JenkinsScript to the shell script section. 14. Give access to jenkins user to execute aws and kubectl commands cp -r /home//.kube /var/lib/jenkins/
systemctl restart jenkins

cd /var/lib/jenkins/.kube  
chown jenkins:jenkins config  
ls -ld config  

sudo -su jenkins  
aws configure  
#aws update-kubeconfig
aws eks update-kubeconfig --name <cluster name> --region <region name>
chmod 777 ~/.kube/config  
15.Execute the job.

Don't forget to do terraform destroy once you are done with this.
