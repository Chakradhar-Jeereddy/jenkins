CI
===
Continoues integration is the process of integrating all the code into single artifcat for VM or image for container.
This should be unit tested. Jenkins job is cloned,built and tested.

Shift-left
==========
Instead of scanning and testing once the development is complted, we can shift/test while development is onging.
Backlog gets reduced. Testing will be done 100s of times, issues gets detected and resolved thoroughly.

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
# Add required dependencies for the jenkins package 
sudo yum install fontconfig java-21-openjdk -y 
sudo yum install jenkins -y
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
lsblk
sudo growpart /dev/nvme0n1 4
lsblk
sudo lvextend -L +10G /dev/mapper/RootVG-homeVol
sudo lvextend -L +10G /dev/mapper/RootVG-rootVol
sudo lvextend -L +10G /dev/mapper/RootVG-varVol
sudo xfs_growfs /
sudo xfs_growfs /home
sudo xfs_growfs /var
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

Approches
====
***Declerative and Scripted pipeline***
- Declarative -> from jenkins 2.0, its DSL(domain specific language on top of groovy).
- This is scrictly evaluated before execution, if errors it will not run.
- It is a wrapper created by jenkins on top of groovy for scrict evaluation.
  
- Scripted pipeline starts with node. It is purely groovy, jenkins will not validate it, it will execute at runtime.
- if error comes in middle, it will stop.

***Hybrid appoach***
- Mixing both declarative and scripted.
```
 stages{
  stage('Build'){                /* put script block in steps */
    steps{
       script{
          sh"""
             echo "Building"
          """
       }
    }
  }
```
Environment Variables
===
- This block is used in pre-build stage
- Anything inside stages is build step
- We use this block before the stages block
- All stages will have access to the variables
```
pipeline{
 agent{
   node{
    label "nodejs"
   }
 }
 environment{
   COURSE = "Jenkins"
 }
 stages{
   stage('Build'){
     steps{
       script{
        sh"""
	     echo "Building"
         echo $COURSE    /* Use the variable inside the stage */
        """
       }
	 }
   }
 }
}
```

Options
===
***Timeout:***
- After timeout Jenkins will abort the pipeline.
- Handle the obort in the post-build section
```
pipeline{
 agent{
   node{
    label "nodejs"
   }
 }
 environment{
   COURSE = "Jenkins"
 }
 options{
  timeout(time: 10, unit: 'SECONDS')  /*Timeout the step if it takes more then 10 seconds. */
 }
 stages{
   stage('Build'){
     steps{
       script{
        sh"""
	     echo "Building"
         echo $COURSE    /* Use the variable inside the stage */
         sleep 12
        """
       }
	 }
   }
 }
 post{
  aborted{
   echo "The stage has been aborted"
  }
 }
}
```
***One build at a time:***
- disableConcurrentBuilds
- Result: Build 9 is already in progress
```
options{
 disableConcurrentBuilds()
}
```

Parameters
===
- After adding parameters, run the build at least once for jekins to execute and undestand.
- During the next run the parameters will be presented.
- Build with parameters
- Accessing the input params using ${params.<variable name>}
```
pipeline{
 agent{
   node{
     label "nodejs"
   }
 }
 parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
 }
 stages{
   stage('build'){
     steps{
      echo "hello"
      echo "Hello ${params.PERSON}"
      echo "Biography: ${params.BIOGRAPHY}"
      echo "Toggle: ${params.TOGGLE}"
      echo "Choice: ${params.CHOICE}"
      echo "Password: ${params.PASSWORD}"
     }
   }
 }
}
```
Triggers
==
- Build periodically (When you want to run jobs in the night
- * * * * * (Every minute)

- GitHub hook trigger for GITScm polling (Jenkins receives a github push hook)
- GitHub hook trigger for GITScm polling
- Git repo -> Settings -> add webhook ->
- Playload URL: http://3.231.58.57:8080/github-webhook/
- content type: application/json
- SSL verification: Disabled
- Which events would you like to trigger this webhook?: Commit or diff

- Poll SCM
- * * * * * -> Every minute
- Every minute jekins pols github to detect changes.

Which is better
==
- Polling is pull mechanism
- Webhook is push mechanism (git will push the event)
- Webhook is better

Put input inside stage
===
```
stage('Deploy'){
      input {
        message "Should we continue?"
        ok "Yes, we should."
        submitter "alice,bob"
        parameters {
          string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        }
      }
      steps{
        script{
          sh"""
          echo "scripted build"
          """
        }
      }
}
```
Check when condition
===
- Use parameter block type boolean to specify if deployment stage is required during the pipeline execution.
- Use WHEN expression in deployment stage and take input from parameters and which decides the execution of the step.
- Parameter in pre-build stage(before stages), When expression should be inside the stage.
```
pipeline{
  agent{
    node{
      label "AGENT"
    }
  }
  environment{
    course = "jenkins"
  }
  options{
    disableConcurrentBuilds()
  }
  parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'Deploy', defaultValue: false, description: 'Do you want to deploy?')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
  }
  stages{
    stage('Build'){
      steps{
        script{
          sh"""
           echo "hello"
          """
        }
      }
    }
    stage('Test'){
      steps{
          sh"""
          echo $course
          """
      }
    }
    stage('Deploy'){
      when{
        expression { "${params.Deploy}" == "true" }
      }
      steps{
        script{
          sh"""
          echo "scripted build"
          """
        }
      }
    }
  }
  post{
    always{
      echo "always say hi"
      cleanWs()
    }
    success{
      echo "say the build successful"
    }
    failure{
      echo "say the build failed"
    }
    aborted{
      echo "timeout exceeded"
    }
  }
}
```
