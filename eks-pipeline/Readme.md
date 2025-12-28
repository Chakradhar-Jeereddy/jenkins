Repo:
==
https://github.com/Chakradhar-Jeereddy/terraform-eks.git

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
