### Working Pipeline 

* A Working Github account
* Jenkins installation.
* Credentials, for DockerHub
* Credentials for Git

```
pipeline {
  environment {
    registry = "nileshc85/cicd-test-1"
    registryCredential = 'jenkins-auth-docker1'
    dockerImage = ''
  }
  
  agent any

    
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }


        stage("Clone Git Repository") {
            steps {
                git(
                    url: "http://gitlab.example.com/root/np-1.git",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }
        
            stage('Building image') {
              steps{
                script {
                  dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
              }
            }
        
        stage('Deploy Image') {
          steps{
             script {
                docker.withRegistry( '', registryCredential ) {
                dockerImage.push()
              }
            }
          }
        }
        
        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
        
        stage('Run Script') {
            steps {
                sh 'chmod +x updatemanifest.sh && ./updatemanifest.sh'
            }
        }

        stage('Git Push') {
            steps {
                sh 'git add .'
                sh 'git commit -m "push to git $BUILD_NUMBER"'
                sh 'git push http://root:haIvVP+K7nXhr0+I8gK6VxYtWFtaWkM7kOkenHhrsGs=@gitlab.example.com/root/np-1.git main'
            }
        }
        
        
    }
}
```
