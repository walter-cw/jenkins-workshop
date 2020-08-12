# Hands-on Jenkins-03 : Creating Jenkins Github Integration on Amazon Linux 2 AWS EC2 Instance
Purpose of the this hands-on training is to teach the students how to **integrate Github with Jenkins jobs**, **create your first Jenkins Pipeline** and **Build a python app with pipeline** on Amazon Linux 2 EC2 instance.

## Learning Outcomes
At the end of the this hands-on training, students will be able to;
- integrate Jenkins Jobs with Github on Amazon Linux 2 EC2 instance
- create Jenkins pipeline
- build a python app with pipeline

## Outline
- Part 1 - Launch Amazon Linux 2 EC2 Instance and Connect with SSH
- Part 2 - Install Jenkins on Amazon Linux 2 EC2 Instance
- Part 3 - Open Jenkins dashboard and create jenkins job with Github integration
- Part 4 - Create Jenkins Pipeline
- Part 5 - Build a python app with pipeline

## Part 1 - Launch Amazon Linux 2 EC2 Instance and Connect with SSH
- Launch an EC2 instance using the Amazon Linux 2 AMI with security group allowing SSH connections.
- Connect to your instance with SSH.
```bash
ssh -i .ssh/call-training.pem ec2-user@ec2-3-133-106-98.us-east-2.compute.amazonaws.com
```
## Part 2 - Install Jenkins on Amazon Linux 2 EC2 Instance using Docker
- Launch an AWS EC2 instance of Amazon Linux 2 AMI with security group allowing SSH and Tomcat 
(8080) ports and name it as `call-jenkins` 
- Connect to the `call-jenkins` instance with SSH.
```bash
ssh -i .ssh/call-training.pem ec2-user@ec2-3-133-106-98.us-east-2.compute.amazonaws.com
```
- Update the installed packages and package cache on your instance.
```bash
sudo yum update -y
```
- Install the most recent Docker Community Edition package.
```bash
sudo amazon-linux-extras install docker -y
```
- Start docker service.
```bash
sudo systemctl start docker
```
- Enable docker service so that docker service can restart automatically after reboots.
```bash
sudo systemctl enable docker
```
- Check if the docker service is up and running.
```bash
sudo systemctl status docker
```
- Add the `ec2-user` to the `docker` group to run docker commands without using `sudo`. 
```bash
sudo usermod -a -G docker ec2-user
```
- Normally, the user needs to re-login into bash shell for the group `docker` to be effective, but `newgrp` command can be used activate `docker` group for `ec2-user`, not to re-login into bash shell.
```bash
newgrp docker
```
- Check the docker version without `sudo`.
```bash
docker version
```
- Run docker jenkins image from Dockerhub Jenkins.
```bash
docker run \
  -u root \
  --rm \
  -d \
  -p 8080:8080 \
  -p 50000:50000 \
  --name jenkins \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```
- Get the administrator password from `var/jenkins_home/secrets/initialAdminPassword` file.
```bash
docker exec -it jenkins bash
cat /var/jenkins_home/secrets/initialAdminPassword
exit
```
- The administrator password can also be taken from docker logs 
```bash
docker logs jenkins
```
- Enter the temporary password to unlock the Jenkins
 
- Install suggested plugins
 
- Create first admin user (call-jenkins:Call-jenkins1234)
 
- Check the URL, then save and finish the installation

## Part 3 - Open Jenkins dashboard and create jenkins job with Github integration

### Part 3/Section 1 - Integration Github with Poll SCM

- Open Jenkins and login to your account.

- Click on "Manage Jenkins"

- Manage plugins 

- There will be a list of all available plugins for Jenkins. See available plugins to install from "Available" tab.

- Search and select "Git plugin" and "GitHub Integration Plugin" then click to "Install without restart". While finding plugins you can use Filter from the top right corner. If these plugins not shown at the search results. They are probably already installed, you can check it from installed tab.

- Plugins will be installed with their dependencies. After the installation has finished, click on to the "Go back to the top page" link.  

- Create a new job and give a name to your job, select "freestyle project" then click "OK". Then you can write a description about your item

- Go down to the <b>Source code management</b> and click on <b>Git</b>.

- We should tell Jenkins where our Git Repository is. So you can give the link to one of your public repositories on GitHub. Please use your own repository.  If your repository is private, you should use your credentials. But in this example, our repository is public so that's not needed. Also, you can tell Jenkins which branch you should build. You can leave it as default. If your all code is in the master branch.

- Then go to Build Triggers and select Poll SCM. You can give instructions that how often and what intervals you want to check the source code repository changes. Based on your requirements, you can give different expressions. For this example, we will check it every minute with "* * * * *" expression.

- You can run your code from **build**. But first you should add your build step. If you are Windows user you can select **Execute Windows batch command**. But if you are using Jenkins from your MacOS or Linux (maybe ec2 Linux machine) you can select **Execute shell**.

- Now we will write our build command. In our example we have simple python code at GitHub and we will call it.
Then **Apply** and **Save** 

- After you saved the project you will see the project menu. From project menu click on **Build Now**. A few seconds later you will see **Build History**. Click on the last build number.

- You can see your Build's result from **console output**. In our example we are printing "Hello from the other side!" with python code. Show students to output.

- Now please go to your GitHub account and change your code. In our example, we changed the inside of the print function from Github or you can change your code from your own machine and push it, it's up to you. Our new code should print "Hello Jenkins Learner!". When we make our change Jenkins will understand the changes in a minute and build it itself. Because we set **Poll SCM** while creating the item. You can see new build number from your build history.

- When you go to last build's console output you can see your changes result.

### Part 3 / Section 2 - Integration Github with GitHub Webhook

- Create a new item

- You can enter the GitHub project.

- Check the GitHub Project from **Source Control Management** then enter your GitHub repository URL.

- Then from Build Triggers click on <b>GitHub hook trigger for GITScm polling</b>

- You can add build steps. But for this example, we want to show you how to trigger GitHub Webhook. So we will not add build command. After that please click on **apply** and **save**

- Go to your **GitHub repository page** then click on **Settings**. 

- You will see **Webhooks** from settings menu. Click on Webhooks.

- Click on **Add webhook**.

- Copy to your Jenkins URL and paste into **Payload URL** section. Be careful, you can't use localhost:8080. But you can use your EC2 machine's Jenkins URL. You must add **/github-webhook/** at the end of your URL. Then click on **Add Webhook**

- Go back to your repository and make changes at your code (make commit).

- When you change your repository. Github webhook understands the changes and immediately tells your Jenkins to update the project. Then Jenkins will automatically start a new build. You will see that the build is started by GitHub.

## Part 4 - Create Jenkins Pipeline

### Part4 / Section 1 - Single Step Pipeline
- We will create our first Jenkins Pipeline **Clarusway_Way to Reinvent Yourself**

- After logging in to Jenkins click on the "New Item" menu option

- Type in the name of the Jenkins Pipeline, Click on the "Pipeline". Then press the "OK" button.

- It will take you directly to the "Configuration" page of the project, select the section called "Pipeline", paste the following code

Jenkinsfile (Declarative Pipeline)

<pre data-editor>
pipeline {
    agent { label 'master' } 
    stages {
        stage('build') {
            steps {
                echo "Clarusway_Way to Reinvent Yourself"
            }
        }
    }
}
</pre>

- Click on the "Build Now" button.

- It will start running the pipeline and within a few seconds, you'll see an indicator of your first job being successful (blue dot on the left side).

- If you click on blue dot on the left side it will take you to the "Console Output"

- Inside the `pipeline` there can be `stages` (We have one). Inside the `stages`, there can be several `stage elements`. 

- Inside each `stage`, there must be `steps`. The steps themselves are Jenkins commands.

- `echo` will just print something on the console. It can be useful for displaying values as the pipeline makes progress.

### Part4 / Section 2 - Running Multiple Steps

- Pipelines are made up of multiple steps that allow you to build, test and deploy applications. Jenkins Pipeline allows you to compose multiple steps in an easy way that can help you model any sort of automation process.

- Think of a step as a single command that does a single action. When a step succeeds it moves onto the next step. When a step fails to execute correctly the Pipeline will fail.

- When all the steps in the Pipeline have successfully completed, the Pipeline is considered successfully executed.

- Create a new pipeline.

- Use this script while  cresating pipeline:

<pre data-editor>
pipeline {
    agent { label 'master' }
    stages {
       stage('build') {
          steps {
             sh 'echo step1'
             sh 'echo step2'
             sh '''
                echo 'Multiline'
                echo 'Example'
             '''
             echo 'not using shell'
          }
       }
    }
}
</pre>

- Save and Apply. Then Build now.

- You will see the multi step pipeline result.

## Part 4 - Build Pipeline with a Python App

- Create a bridge network in Docker using the following docker network create command:

<pre data-editor="" data-theme="dawn" data-border-color="darkslategray" data-readonly="">
$ docker network create jenkins
470a40abd7f2e06b22886827f162b3f1db7ea2fd535b242326b82482fd42b205 
</pre>

- Create the following volumes to share the Docker client TLS certificates needed to connect to the Docker daemon and persist the Jenkins data using the following docker volume create commands:

<pre data-editor="" data-theme="dawn" data-border-color="darkslategray" data-readonly="">
$ docker volume create jenkins-docker-certs
jenkins-docker-certs
$ docker volume create jenkins-data
jenkins-data
</pre>

- In order to execute Docker commands inside Jenkins nodes, download and run the **docker:dind** Docker image using the following docker container run command:

<pre data-editor="" data-theme="dawn" data-border-color="darkslategray" data-readonly="">
$ docker container run --name jenkins-docker --rm --detach \
   --privileged --network jenkins --network-alias docker \
   --env DOCKER_TLS_CERTDIR=/certs \
   --volume jenkins-docker-certs:/certs/client \
   --volume jenkins-data:/var/jenkins_home \
   --volume "$HOME":/home docker:dind
</pre>
 
- Run the jenkinsci/blueocean image as a container in Docker using the following docker container run command (bearing in mind that this command automatically downloads the image if this hasn’t been done):

<pre data-editor="" data-theme="dawn" data-border-color="darkslategray" data-readonly="">
docker container run --name jenkins-tutorial --rm --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --volume "$HOME":/home --publish 8080:8080 jenkinsci/blueocean
</pre>

* Maps the /var/jenkins_home directory in the container to the Docker volume with the name jenkins-data. If this volume does not exist, then this docker container run command will automatically create the volume for you.

* Maps the $HOME directory on the host (i.e. your local) machine (usually the /Users/<your-username> directory) to the /home directory in the container.
