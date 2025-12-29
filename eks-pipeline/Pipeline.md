Github Repo:
==
https://github.com/Chakradhar-Jeereddy/terraform-eks.git

1. In jekins, create a folder INFRA.
2. Inside folder create New item/Job (type pipleline and source git).
3. Specify the Jenkinsfile **00-vpc/Jenkinsfile**.
4. Create S3 bucket with same name that exists in provider.tf.
5. Install AWS credentials and create credentials using Acess key and secret key.
6. Install terrafrom in Jenkins agent
```
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

**Jenkinsfile:** Remove the comments while executing it.
```
pipeline {
    // These are pre-build sections
    agent {
        node {
            label 'agent'
        }
    }
    environment {
        COURSE = "Jenkins"
    }
    options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    stages {
        stage('Init') {
            steps {
                script{
                    withAWS(region:'us-east-1',credentials:'aws-ecr') {          /* It needs aws credentials to access S3 */
                        sh """
                            cd 00-vpc
                            terraform init
                        """
                    }
                }
            }
        }
        stage('Plan') {
            steps {
                script{
                    withAWS(region:'us-east-1',credentials:'aws-ecr) {   /* It needs aws credentials to access S3 */
                        sh """
                            cd 00-vpc
                            terraform plan
                        """
                    }
                }
            }
        }
        stage('Deploy') {      /* Asks for approval */
            input {
                message "Should we continue?"
                ok "Yes, we should."
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Chakradhar', description: 'Who should I say hello to?')
                }
            }
            
            steps {
                script{
                    withAWS(region:'us-east-1',credentials:'aws-ecr') {  /* It needs credentials to build infra */
                        sh """
                            cd 00-vpc
                            terraform apply -auto-approve
                        """
                    }
                }
            }
        }
    post{
        always{
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success {
            echo 'I will run if success'
        }
        failure {
            echo 'I will run if failure'
        }
        aborted {
            echo 'pipeline is aborted'
        }
    }
}
```

Install plugin ansiColor, for colors to work
==
1. Jenkins -> Manage -> Ansicolor
2. Restart jenkins -> sudo systemctl restart jenkins
3. Use the function **ansiColor('xterm')** in the options block
4. options block always comes before stages.

```
    options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

## The below out will be shown in green color

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

```
   
Trigger downstream pipeline
===

- If 00-vpc is successful trigger 10-SG pipeline. 00-vps -> 10-SG
- **10-SG:** For creating security groups.
- Create a new pipeline under same infra folder with name 10-SG
- Copy from -> 00-vpc (so that we can copy the pipeline settings from upstream, like repo URL, etc.)
- copy the jeknisfile from 00-vpc to 10-sg and change the directy name to 00-sg
- Add the below code in jenkinsfile of 00-vpc(upstream) so it can trigger the job.
**Code:**
```
stage('Trigger Parameterized Job') {
    steps {
        script {
              build job: 'deployment_job',
              propagate: false,   /* Should we mark the status of this job as failure when upstream fails? */
              wait: false  /* Should we wait till the upstream job is complete? */
        }
    }
}
```
