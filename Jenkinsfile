pipeline{
    agent{
        label 'Jenkins-Agent'
    }
    tools{
        maven 'Maven3'
        jdk 'Java17'
    }
    environment{
        APP_NAME = 'register-app-pipeline'
        RELEASE = '1.0.0'
        DOCKER_USER = 'devopswthaws58'
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JOB_NAME= 'User-Register-Application-CI'
    }
    stages{
        stage('cleanup workspace'){
            steps{ 
                cleanWs()
            }
        }
        stage('checkout from SCM'){
            steps{
                git branch: 'main', 
                    credentialsId: 'github', 
                    url: 'https://github.com/Devopswithaws58/register-app.git'
            }
        }
        stage('build application'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('test application'){
            steps{
                sh 'mvn test'
            }
        }
        stage('sonarqube analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage('quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
       }
       stage('Trivy scan the docker image'){
        steps{
            script{
                 sh ('docker run --rm -v "/$(pwd)/var/run/docker.sock:/var/run/docker.sock" aquasec/trivy image devopswthaws58/register-app:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
       }
       stage('cleanup artifacts'){
        steps{
            script{
                sh 'docker rmi ${IMAGE_NAME}:${IMAGE_TAG}'
                sh 'docker rmi ${IMAGE_NAME}:latest'
                }
            }
       }
       stage('trigger CD Pipeline'){ 
        steps{
            script{
                buildTokenTrigger credentialsId: 'token', jenkinsUrl: 'http://ec2-13-127-175-27.ap-south-1.compute.amazonaws.com:8080', job: 'GitOps-Register-Application-CD', parameters: [IMAGE_TAG: IMAGE_TAG]
                }
            }
       }
    }
    post {
       failure {
             emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                      subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                      mimeType: 'text/html',to: "devopswithaws58@gmail.com"
      }
      success {
            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                     subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                     mimeType: 'text/html',to: "devopswithaws58@gmail.com"
      }      
   }
}
