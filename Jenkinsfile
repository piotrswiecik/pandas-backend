def imageName="192.168.56.200:8082/docker_registry/backend"
def dockerRegistry="https://192.168.56.200:8082"
def registryCredentials="artifactory"
def dockerTag=""

pipeline {
    agent {
        label 'agent'
    }
    environment {
        scannerHome = tool 'SonarQube'
    }
    
    stages {
        
     // git checkout stage
      stage('pull') {
        steps {
          sh 'echo "git pull stage"'
          git branch: 'main', url: 'https://github.com/piotrswiecik/pandas-backend.git'
        checkout scm
          sh 'ls -al'
        }
      }

    // pytest stage
      stage('test') {
        steps {
          sh 'python3 -m pip install -r requirements.txt'
          sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
        }
      }
      
      // sonar stage
      stage('sonar') {
        options {
          timeout(time: 1, unit: 'MINUTES')
        }
        steps {
          sh 'echo "sonarqube stage"'
          withSonarQubeEnv('panda-sonar') {
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
      
      // docker build stage
      stage('docker build') {
          steps {
              script {
                  dockerTag = "RC-${env.BUILD_ID}.test"
                  applicationImage = docker.build("$imageName:$dockerTag", ".")
              }
          }
      }
      
      // docker push stage
      stage('docker push') {
        steps {
            script {
                docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                    applicationImage.push()
                    applicationImage.push("latest")
                }
            }
        }
      }

      // push to repo
      stage ('Push to repo') {
            steps {
                dir('ArgoCD') {
                    withCredentials([gitUsernamePassword(credentialsId: 'git', gitToolName: 'Default')]) {
                        git branch: 'master', url: 'https://github.com/piotrswiecik/pandas-argocd.git'
                        sh """ cd backend
                        git config --global user.email "piotr.swiecik@gmail.com"
                        git config --global user.name "ps"
                        sed -i "s#$imageName.*#$imageName:$dockerTag#g" backend-deployment.yaml
                        git commit -am "Set new $dockerTag tag."
                        git diff
                        git push origin master
                        """
                    }                  
                } 
            }
        }
      
    // end of stages  
    }
   
}