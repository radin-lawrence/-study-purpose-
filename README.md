# How-to-push-docker-image-to-Docker-hubusing-jenkins-pipeline

In this article we are building a Docker image and deploying it into Docker hub using Jenkins. We use multi agent setup for setting up our project.Here we use Jenkins pipeline to automate docker build and push to docker hub:

## steps:

Steps to perform Jenkins pipeline to build and push docker image:


  1. Clone flaskApp Project from Github. You can get my [FlaskApp Git](https://github.com/radin-lawrence/devops-flask)
  2. Build docker image using jenkins
  3. Login to docker hub
  4. Push image to docker hub
  5. create a conatiner from build image
  6. Log out from docker hub

## Pre-requisites:

  1. Jenkins master server with Git
  2. Build server (Docker installed)
  3. Test server (Docker installed)
  4. Docker hub account

## Procedure:

### Step 1: Install Jenkins

We need to install jenkins for our master jenkins server, for installing jenkins:

~~~sh
amazon-linux-extras install epel -y
yum install java-1.8.0-openjdk-devel -y
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins -y
systemctl start jenkins.service
systemctl enable jenkins.service
~~~

Once Jenkins is installed, we can use this link for accesing jenkins: http://publicIPofserver:8080.

It will prompt for password token which can be fetched from: 
~~~sh
cat /var/lib/jenkins/secrets/initialAdminPassword
~~~

> We need to also install git to master server:

~~~sh
yum install git -y
~~~

### Step 2: Adding Docker hub to jenkins

Here, we add Docker Hub credentials into jenkins to get login access without passing credentials in script. 
For adding Docker hub credential in Jenkins follow below steps:

First we need to generate Dockerhub account credentials by this [document](https://docs.docker.com/docker-hub/access-tokens/)
Copy the token generated under dockerhub 

Now under jenkins we need to add this generated token to combine [Docker hub](https://www.jenkins.io/doc/book/using/using-credentials/)

![image](https://user-images.githubusercontent.com/100773863/170074183-b308a913-ad99-42e2-8cf0-3795bd1adb7c.png)

Add the docker hub username and password token that generated at Docker Hub.

### Step 3: Create Build and Test Agent nodes

  1. Go to your Jenkins dashboard
  2. Go to Manage Jenkins option in main menu
  3. Go to Manage Nodes and clouds item
  4. Go to New Node option in side menu
  5. Fill the Node/agent name and select the type; (e.g. Name: build and test, Type: Permanent Agent)
  6. Now fill the fields for build agent server:
     ![image](https://user-images.githubusercontent.com/100773863/170077317-ddbe8900-0847-4c7a-a420-469596a82019.png)
     ![image](https://user-images.githubusercontent.com/100773863/170077568-1f8e1f48-f545-41eb-8c9f-07aa68cd92d3.png)

  > While filling these details, please fill as above image because we need to set some values that are needed for our project. In our project we call agents using labels, so we need to select this: 
  ![image](https://user-images.githubusercontent.com/100773863/170082912-8374c823-039f-4d5c-88fa-23c3a10fe65d.png)
  > Also, in above screenshot of my build agent there is an option "credentials", in there we need to add our build server username and password.
  
  7. Now we need to enable password authentication on both machines (/etc/ssh/sshd_config) and restart the SSH service on both agent servers.
  8.  Do the same (step 6 and 7) for Test agent server as well, but use their own private IPs on host section.
  9.  Once those are added, save both nodes

***Now we need to generate an ssh public key using ssh-keygen in master and copy it to both Build and Test agent servers***
  ~~~sh
  # ssh-keygen
  ~~~
  The identification and public key has been saved in /root/.ssh/id_rsa. & /root/.ssh/id_rsa.pub.
  
  10. Now we need to copy the SSH key ID to both servers using:
  ~~~sh
  ssh-copy-id -i root@192.168.0.106  --->private ip of agent servers
  ~~~
  Do for both servers (Build and Test)
  11. We need to install docker on both agent node servers:
  ~~~sh
  yum install docker -y
  ~~~
  12. To launch a node agents we need java installed on the agent machines:
  ~~~sh
  yum install java -y
  ~~~
  13. Now click on lauch agent option in jenkins where we created 2 agent nodes.

![image](https://user-images.githubusercontent.com/100773863/170088962-45e5dbc3-4b3e-450e-80ac-e827e37ad67f.png)

  
### Step 4: Add Pipeline script
 Now we need to create pipeline script for our project:
 1. Click the New Item menu within Jenkins Dashboard
 2. Provide a name for your new item (e.g. flaskapp) and select Pipeline:
 ![image](https://user-images.githubusercontent.com/100773863/170083467-342962be-632e-4ca0-b185-d06993e668f4.png)
 3. Add the necessary details in description
 4. Now under section "Pipeline" add our gruvy script:
 ~~~sh
pipeline{
	agent { label('build || test') }
	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}
	stages {
	    stage('gitclone') {
	        agent {
                  label 'build'
                  }
			steps {
				git 'https://github.com/radin-lawrence/devops-flask'
			}
		}
		stage('Build') {
            agent {
                  label 'build'
                  }
			steps {
				sh 'docker build -t radinlawrence/flaskapp:version1 .'
			}
		}
		stage('Login') {
            agent {
                  label 'build'
                  }
			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
		stage('Push') {
            agent {
                  label 'build'
                  }
			steps {
				sh 'docker push radinlawrence/flaskapp:version1'
			}
		}
		stage('Run docker container'){
		    agent {
                  label 'test'
                  }
            steps {
                sh 'docker run -p 80:5000 -d --name webserver radinlawrence/flaskapp:version1'
            }
		}
	}
	post {
	     always {
		     	sh 'docker logout'
		}
	}
}
 ~~~
The script will look like:

![image](https://user-images.githubusercontent.com/100773863/170084303-dc1786a2-ada1-494b-ac12-bcc72a1f84b7.png)

### Step 5: Build our project

Once these are completed we can run our project by clicking Build now option in our project, once the build is complete it will looks like:
![image](https://user-images.githubusercontent.com/100773863/170086549-445e455f-b63e-4ab3-ba7c-15b517555261.png)




## Conclusion

In this article, we build a docker image and pushed it to docker hub using jenkins pipeline. Please contact me if you have any questions in this section. Thank you!



