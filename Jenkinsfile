pipeline{
    agent{
        label 'Jenkins-Agent'
    }
    tools{
        maven 'Maven3'
        jdk 'Java17'
    }
    environment{
        APP_NAME = 'register-app'
        RELEASE = '1.0.0'
        DOCKER_USER = 'devopswthaws58'
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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
                sh 'curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG:${IMAGE_TAG}' 'http://ec2-3-108-55-35.ap-south-1.compute.amazonaws.com:8080/job/GitOps-register-app-cd/buildWithParameters?token=gitops-token''
                }
            }
       }
    }
}
