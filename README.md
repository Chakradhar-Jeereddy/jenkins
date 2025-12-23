CI
===
Continoues integration is the process of integrating all the code into single artifcat for VM or image for container.
This should be unit tested, for every commit, the CI jobs is called. The job will cloned,build and tested.

Shift-left
==========
Instead of scanning and testing one the development is complted, we can shift/test while development is onging.
Backlog will reduce. Testing will be done 100s of times, issues will be detected and resolved thoroughly.

- CI Tools

Jenkins and Github Actions
===
Jenkins is a modular service, its power lies in plugins. If you install plugins, Jenkins can connect to that tools.
It gives us nice interface, control, logging, scheduling, RBAC, auditing, webhook, shared library/pipelines .

Install Jenkins
==
https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos
```
sudo curl -o /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
# Add required dependencies for the jenkins package 
sudo yum install fontconfig java-21-openjdk -y 
sudo yum install jenkins
sudo systemctl daemon-reload
sudo systemctl start jenkins
 sudo systemctl enable jenkins
netstat -nltp # check port 8080
```

- After installing jenkins 
- Open the console using IP:8080
- Enter the default admin password from
- /var/lib/jenkins/secrets/initialAdminPassword for first time login
- Install suggested plugins

*** In jenkins everything is a job***

Types of jobs
============
Freestyle jobs
Pipelines

Testing
===
- Create job
- Name: hello-world
- Freestyle project -> OK
- Build steps -> Add biuld step -> execute shell command
- echo "Hello World" -> save
- Click build now
- Click the running job and console output to see log.
  
Disadvantages of freestyle
===
- No version control like git
- Cannot track changes
- Cannot revert to earlier stage
- Tough to do changes
- Cannot verify (if someone deletes the scripts)

Pilepline creation
===
- Click New item
- Name: hello-pipeline
- pipeline -> pipeline script from SCM(source code managment) -> sample -> hello world
- Jekins pipeline syntax -> https://www.jenkins.io/doc/book/pipeline/
```
pipeline {  
    agent{
       node{
         label 'AGENT-1' Add this label.
       }
    }
    stages {
        stage('Build') {  ## Build state
            steps {
                echo "Building"
            }
        }
        stage('Test') {  ## Testing stage
            steps {
               echo "Testing"
            }
        }
        stage('Deploy') {  ## Deployment stage
            steps {
               echo "Deploying"
            }
        }
    }
}
```
- Create a rep jenkins and a file Jenkinsfile (Upper case J)
- Put the above code and save file
- In Jenkins job configuration, select SCM: git, repo URL, no creds and branch: main
- Script path: Jenkinsfile -> save
- Click build now


Add plugin state view from available plugins
===
- Pipeline: Stage View
- Install
- run build again, you can see the stages in UI.

Master and node
===
- Jenkins master can't handle all the loads
- Single server can't have different OS to test
- We create multiple agents for different purpose and run our jobs there
- Master will co-ordinate with the agents
- Master connects to node via SSH
- Java should be installed in agent.

Create node
===
- EC2 instance with name jenkins-agent
- size: t2.small
- AMI: Devops practise
- disk 50gb
- resize disk
```
sudo -i
lsblk
growpart /dev/nvme0n1 4
lsblk
Add space to /tmp which is root and home
sudo lvextend -L +20G /dev/mapper/RootVG-homeVol
sudo lvextend -L +10G /dev/mapper/RootVG-rootVol
sudo xfs_growfs /
sudo xfs_growfs /home
df -h
```
- Install Java in the agent server
- On consol, Manage jenkins -> nodes -> new node
- Name: AGENT-1
- Add permenent agent
- Number of executors: 3   (3 jobs can run at same time)
- Root directory: /home/ec2-user/jenkins
- Labels: AGENT-1  (we give nodejs, python based on type of node)
- Usage: only build jobs with label expression
- Launch Method: Launch agents via ssh
- Agent host: 172.31.0.53  (private ip)
- Credentials for ssh : ec2-user
- Availability: Keep this agent online as much as possible.
- Host key verification: non verify and save.
- Once the agent is added and showing online.
- Added agent to the pipeline in Jenkinsfile, othrwise it goes to master
```
agent {
    node {
        label 'my-defined-label'
        customWorkspace '/some/other/path'
    }
}
```
- Run the build again, the job will run on agent.

Defaut workspace
===
- /var/lib/jenkins

Pipeline stages
===
- Pre-build -> How to trigger, env setup
- Build  -> What to do?
- Post-build -> After completion

Post
====
***Always:*** Run the steps in the post section regardless of the completion status of the Pipeline’s or stage’s run.
```
    post { 
        always { 
            echo 'I will always say Hello again!'
        }
    }
```

Clean Workspace
===
```
    post {
        always {
            cleanWs() /* Cleans the workspace after every build */
        }
    }
```
```
pipeline{
 agent{
   node{
    label "nodejs"
   }
 }
 stages{
   stage('Build'){
     steps{
	   echo "Building"
	   }
   }
   stage('Test'){
     steps{
	   echo "Testing"
	   }
   }
   stage('Deploying'){
     steps{
	   echo "Deploying"
	   }
   }
 }
 post{
  always{
     echo "Always say Hi and Hello"
     cleanWs()
  }
  success{
   echo "I will run if failure"
  }
  failure{
   echo "I will run if success"
  }
 }
}
```









Select built with parameters
Name: COMPONENT
Execute shell - 
ansible-playbook roboshop.yml -i inv  -e ansible_user=centos 
-e ansible_password=DevOps321 -e role_name=${COMPONENT} -
e hosts=$(echo $COMPONENT | tr [a-z] [A-Z])

ansible-playbook roboshop.yml -i inv  -e ansible_user=centos 
-e ansible_password=DevOps321 -e role_name=${COMPONENT} 
-e hosts=$(echo $COMPONENT | tr [a-z] [A-Z]) 
--ssh-extra-args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'

Ansible configuration file (Sample)
https://github.com/ansible/ansible/blob/stable-2.11/examples/ansible.cfg
copy the content and create a config file in your project
To enable color
force_color = True (default is false)
In jenkins
Build Environment -> select color ANSI console output
