
pipeline {
    agent {
        label "jenkin-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
    APP_NAME = "complete-prodcution-e2e-pipeline"
    RELEASE = "1.0.0"
    DOCKER_USER = "awssoorinje"
    DOCKER_PASS = 'awssoorinje'
    IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }


    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("checkout from ssh") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/soorinjeaws/complete-prodcution-e2e-pipeline.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("sonar-test") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality-Gate") {
            steps {
                script {
                    sleep(time: 10, unit: 'SECONDS')
                    waitForQualityGate abortPipeline: false, credentialsId:'sonar-token'
                } 
            }
        }
        stage("Docker-Build-Push") {
            steps {
                script {
                   docker.withRegistry('',DOCKER_PASS){
                      docker_image = docker.build "${IMAGE_NAME}"
                   }

                   docker.withRegistry('',DOCKER_PASS) {
                     docker_image.push("${IMAGE_TAG}")
                     docker_image.push('latest')
                   }
                } 
            }
        }
         stage("trigger-cd-pipleline") {
            steps {
                script {
                   sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkin-server.techcont.com/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
                } 
            }
        }
    }
}
