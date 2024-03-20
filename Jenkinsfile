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
        IMAGE_NAME = '${DOCKER_USER}' + '/' + '${APP_NAME}'
        IMAGE_TAG = '${RELEASE}-${BUILD_NUMBER}'
    }
    stages{
        stage('cleanup workspace'){
            steps{ 
                cleanWs()
            }
        }
        stage('checkout from SCM'){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Devopswithaws58/register-app.git'
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
    }
}
