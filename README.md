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

Types of jobs
============
Freestyle jobs
Pipelines

- Create job
- Name: hello-world
- Freestyle project -> OK
- 
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
