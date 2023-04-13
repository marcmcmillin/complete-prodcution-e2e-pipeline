pipeline{
    agent {
        label "jenkins-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                script {
                    cleanWs()
                }
            }
        }

        stage("Checkout Code") {
            steps {
                script {
                    git branch: 'main', credentialsId: 'github-pat', url: 'https://github.com/dmancloud/complete-devops-pipeline'
                }
            }        
        }

        stage("Build Application") {
            steps {
                script {
                    sh "mvn clean package"
                }
            }        
        }

        stage("Test Application") {
            steps {
                script {
                    sh "mvn test"
                }
            }        
        }

        stage("Static Code Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }        
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }        
        }

    }
}