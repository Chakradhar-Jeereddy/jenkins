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
sudo curl -O /etc/yum.repos.d/jenkins.repo \
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
    agent any  # which worker node
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


Add plugin state view from available plugins
===
- Pipeline: Stage View
- Install
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
