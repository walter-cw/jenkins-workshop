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
### Part 4 / Section 1 Install Docker & Jenkins

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
  
### Part 4 / Section 2 - Fork and clone the sample repository on GitHub

- Obtain the simple "add" Python application from GitHub, by forking the sample repository of the application’s source code into your own GitHub account and then cloning this fork locally.

- Ensure you are signed in to your GitHub account. 

- Fork the [https://github.com/jenkins-docs/simple-python-pyinstaller-app] (simple-python-pyinstaller-app) on GitHub into your local GitHub account. If you need help with this process, refer to the [https://help.github.com/en/github/getting-started-with-github/fork-a-repo](Fork A Repo) documentation on the GitHub website for more information.

- Clone your forked `simple-python-pyinstaller-app repository` (on GitHub) locally to your machine. To begin this process, do either of the following (where <your-username> is the name of your user account on your operating system):

- Open up a terminal/command line prompt and cd to the appropriate directory on:

 `home/ec2-user/GitHub`

- Run the following command to continue/complete cloning your forked repo:
<pre>
git clone https://github.com/YOUR-GITHUB-ACCOUNT-NAME/simple-python-pyinstaller-app</pre>
where YOUR-GITHUB-ACCOUNT-NAME is the name of your GitHub account.

### Part 4 / Section 3 - Create your Pipeline project in Jenkins

- We can access our jenkins in ec2 instance IPv4 Public IP:8080

- To access the admin password, execute the following commands:
<pre>
$ docker container exec -it jenkins-tutorial bash
$ cat /var/jenkins_home/secrets/initialAdminPassword
</pre>

- After you log in to Jenkins,  click create new jobs under Welcome to Jenkins!
*Note: If you don’t see this, click New Item at the top left.* 

- In the Enter an item name field, write a name for your new Pipeline project.

- Scroll down and click Pipeline, then click OK at the end of the page.

- ( Optional ) On the next page, specify a brief description for your Pipeline in the Description field (e.g. An entry-level Pipeline demonstrating how to use Jenkins to build a simple Python application)

- Click the Pipeline tab at the top of the page to scroll down to the Pipeline section.

- From the Definition field, choose the Pipeline script from SCM option. This option instructs Jenkins to obtain your Pipeline from Source Control Management (SCM).

- From the SCM field, choose Git.

- In the Repository URL field, specify the directory path of your locally cloned repository above, which is from your user account/home directory on your host machine, mapped to the /home directory of the Jenkins container - i.e.

(For ec2 instance: `/home/GitHub/simple-python-pyinstaller-app)`

- Click Save to save your new Pipeline project. You’re now ready to build.

### Part 4 / Section 4 - The Jenkinsfile

- We’re now ready to create your Pipeline that will automate building our Python application. Our Pipeline will be created as a Jenkinsfile, which will be committed to our locally cloned Git repository (simple-python-pyinstaller-app).

- This is the foundation of "Pipeline-as-Code", which treats the continuous delivery pipeline a part of the application to be versioned and reviewed like any other code. 

- First, create an initial Pipeline with a "Build" stage that executes the first part of the entire production process for our application. This "Build" stage downloads a Python Docker image and runs it as a Docker container, which in turn compiles your simple Python application into byte code.

- Using a text editor, create and save new text file with the name Jenkinsfile at the root of your local simple-python-pyinstaller-app Git repository.

- Copy the following Declarative Pipeline code and paste it into your empty Jenkinsfile:
 
The Jenkinsfile in the repository is 
<pre>
pipeline {
    agent none       //(1)
    stages {
        stage('Build') {       //(2)
            agent {
                docker {
                    image 'python:2-alpine'       //(3)
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'       //(4)
                stash(name: 'compiled-results', includes: 'sources/*.py*')       //(5)
            }
        }
    }
}
</pre>

(1) The agent section with the none parameter specified at the top of this Pipeline code block means that no global agent will be allocated for the entire Pipeline’s execution and that each stage directive must specify its own agent section.

(2) Defines a stage (directive) called Build that appears on the Jenkins UI.
This image parameter (of the agent section’s docker parameter) downloads the python:2-alpine Docker image (if it’s not already available on your machine) and runs this image as a separate container. This means that:

(3)

- You’ll have separate Jenkins and Python containers running locally in Docker.

- The Python container becomes the agent that Jenkins uses to run the Build stage of your Pipeline project. However, this container is short-lived - its lifespan is only that of the duration of your Build stage’s execution.

(4) This sh step (of the steps section) runs the Python command to compile your application and its calc library into byte code files (each with .pyc extension), which are placed into the sources workspace directory (within the /var/jenkins_home/workspace/simple-python-pyinstaller-app directory in the Jenkins container).

(5) This stash step (of the basic steps section) saves the Python source code and compiled byte code files (with .pyc extension) from the sources workspace directory for use in later stages.

- Save your edited Jenkinsfile and commit it to your local simple-python-pyinstaller-app Git repository.

`git commit -m "Add initial Jenkinsfile"` 

- Go back to Jenkins again, log in again if necessary and click Open Blue Ocean on the left to access Jenkins’s Blue Ocean interface.

- In the This job has not been run message box, click Run, then quickly click the OPEN link which appears briefly at the lower-right to see Jenkins running your Pipeline project. If you weren’t able to click the OPEN link, click the row on the main Blue Ocean interface to access this feature.

>> You may need to wait a few minutes for this first run to complete. After making a clone of your local simple-python-pyinstaller-app Git repository itself, Jenkins:

  a. Initially queues the project to be run on the agent.<br>
  b. Runs the Build stage (defined in the Jenkinsfile) on the Python container. During this time, Python uses the py_compile module to compile the code of your Python application and its calc library into byte code, which are stored in the sources workspace directory (within the Jenkins home directory).

The Blue Ocean interface turns green if Jenkins compiled your Python application successfully.

6. Click the X at the top-right to return to the main Blue Ocean interface.

### Part 4 / Section 5 - Add a test stage to your Pipeline

- Go back to your text editor and open your Jenkinsfile.

- Copy and paste the following Declarative Pipeline syntax immediately under the Build stage of your Jenkinsfile:

.<pre>
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
</pre>

so that you end up with:

<pre>
pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {       //(1) 
            agent {
                docker {
                    image 'qnib/pytest'       //(2) 
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'       //(3)  
            }
            post {
                always {
                    junit 'test-reports/results.xml'       //(4) 
                }
            }
        }
    }
}
</pre>

(1) Defines a stage (directive) called Test that appears on the Jenkins UI.

(2) This image parameter (of the agent section’s docker parameter) downloads the qnib:pytest Docker image (if it’s not already available on your machine) and runs this image as a separate container. This means that:

- You’ll have separate Jenkins and pytest containers running locally in Docker.

- The pytest container becomes the agent that Jenkins uses to run the Test stage of your Pipeline project. This container’s lifespan lasts the duration of your Test stage’s execution.

(3) This sh step (of the steps section) executes pytest’s py.test command on sources/test_calc.py, which runs a set of unit tests (defined in test_calc.py) on the "calc" library’s add2 function (used by your simple Python application add2vals). The:

- --junit-xml test-reports/results.xml option makes py.test generate a JUnit XML report, which is saved to test-reports/results.xml (within the /var/jenkins_home/workspace/simple-python-pyinstaller-app directory in the Jenkins container).

(4) This junit step (provided by the JUnit Plugin) archives the JUnit XML report (generated by the py.test command above) and exposes the results through the Jenkins interface. In Blue Ocean, the results are accessible through the Tests page of a Pipeline run. The post section’s always condition that contains this junit step ensures that the step is always executed at the completion of the Test stage, regardless of the stage’s outcome.

- Save your edited Jenkinsfile and commit it to your local simple-python-pyinstaller-app Git repository.

- Go back to Jenkins again, log in again if necessary and ensure you’ve accessed Jenkins’s Blue Ocean interface.

- Click Run at the top left, then quickly click the OPEN link which appears briefly at the lower-right to see Jenkins running your amended Pipeline project. If you weren’t able to click the OPEN link, click the top row on the Blue Ocean interface to access this feature.

- Click the X at the top-right to return to the main Blue Ocean interface.

### Part 4 / Section 6 - Add a final deliver stage to your Pipeline

- Go back to your text editor and open your Jenkinsfile.

- Copy and paste the following Declarative Pipeline syntax immediately under the Test stage of your Jenkinsfile:

<pre>
        stage('Deliver') {       
            agent any
            environment {       
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) {       
                    unstash(name: 'compiled-results')       
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"       
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
</pre>

and add a `skipStagesAfterUnstable` option so that you end up with:

<pre>
pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {      //(1) 
            agent any
            environment {       //(2) 
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) {       //(3) 
                    unstash(name: 'compiled-results')       //(4) 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"       //(5) 
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"       //(6) 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}
</pre>

(1) Defines a stage (directive) called Deliver that appears on the Jenkins UI.

(2) This environment block defines two variables which will be used later in the 'Deliver' stage.

(3) This dir step (of the basic steps section) creates a new subdirectory named by the build number. The final program will be created in that directory by pyinstaller. BUILD_ID is one of the pre-defined Jenkins environment variables and is available in all jobs.

(4) This unstash step (of the basic steps section) restores the Python source code and compiled byte code files (with .pyc extension) from the previously saved stash. image] (if it’s not already available on your machine) and runs this image as a separate container. This means that:

>> You’ll have separate Jenkins and PyInstaller (for Linux) containers running locally in Docker.

>> The PyInstaller container becomes the agent that Jenkins uses to run the Deliver stage of your Pipeline project. This container’s lifespan lasts the duration of your Deliver stage’s execution.

(5) This sh step (of the steps section) executes the pyinstaller command (in the PyInstaller container) on your simple Python application. This bundles your add2vals.py Python application into a single standalone executable file (via the --onefile option) and outputs the this file to the dist workspace directory (within the Jenkins home directory). Although this step consists of a single command, as a general principle, it’s a good idea to keep your Pipeline code (i.e. the Jenkinsfile) as tidy as possible and place more complex build steps (particularly for stages consisting of 2 or more steps) into separate shell script files like the deliver.sh file. This ultimately makes maintaining your Pipeline code easier, especially if your Pipeline gains more complexity.

(6) This archiveArtifacts step (provided as part of Jenkins core) archives the standalone executable file (generated by the pyinstaller command above at dist/add2vals within the Jenkins home’s workspace directory) and exposes this file through the Jenkins interface. In Blue Ocean, archived artifacts like these are accessible through the Artifacts page of a Pipeline run. The post section’s success condition that contains this archiveArtifacts step ensures that the step is executed at the completion of the Deliver stage only if this stage completed successfully.

- Save your edited Jenkinsfile and commit it to your local simple-python-pyinstaller-app Git repository.

- Go back to Jenkins again, log in again if necessary and ensure you’ve accessed Jenkins’s Blue Ocean interface.

- Click Run at the top left, then quickly click the OPEN link which appears briefly at the lower-right to see Jenkins running your amended Pipeline project. If you weren’t able to click the OPEN link, click the top row on the Blue Ocean interface to access this feature.

- Click the X at the top-right to return to the main Blue Ocean interface, which lists your previous Pipeline runs in reverse chronological order.

