pipeline{
    agent{
        label "jenkin-agent"
    }
    tools {
        jdk 'Java17'
        maven  'Maven3'
    }
    stages{
        stage("Cleanup  Workspace"){
            steps {
                cleanWs()
            }
        }
    
        stage("checkout from ssh"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/soorinjeaws/complete-prodcution-e2e-pipeline.git'  
            }
        }

        stage("Build Application"){
            steps{
                sh  "mvn clean package"      
            }
        }

        stage("Test Application"){
            steps{
                sh  "mvn test"      
            }
        }

        stage("sonar-test"){
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                    sh "mvn sonar:sonar"
                    }
                } 
            }
        }
        stage("Qaulity-Date"){
            steps{
                script {
                    waitForQualityGate abortPipeline: false, credentialsId:'sonar-token'
                } 
            }
        }



    }
}
