pipeline {
    agent any
    stages {
        stage('Build') {         # this will run first
            steps {
                echo "Building"
            }
        }
        stage('Test') {            # this will run next
            steps {
                echo "Testing"
            }
        }
        stage('Deploy') {        # This will run last
            steps {
                echo "Deploying"
            }
        }
    }
}

