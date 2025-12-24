Follow catalogue documentation:
===
 https://github.com/daws-86s/roboshop-documentation/blob/main/02-catalogue.MD
- Install required softwares on Jenkin's agent to build the catalogue image
```
dnf module disable nodejs -y
dnf module enable nodejs:20 -y
dnf install nodejs -y
```
- Update the version in package.json file.
-   "version": "1.1.0"
-   We need to read this version, need a plugin to read package json in jenkins pipeline
-   def packageJson = readJSON file: 'package.json'
-   Need pulgin ***Pipeline Utility Steps*** to get the above function.
```
pipeline{
  agent{
    node{
      label "AGENT"
    }
  }
  environment{
    course = "jenkins"
    appVersion = ""
  }
  options{
    disableConcurrentBuilds()
  }
  stages{
    stage('Read Version'){
      steps{
        script{
            def packageJSON = readJSON file: 'package.json'
            appVersion = packageJSON.version
            echo "app version: ${appVersion}"
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
