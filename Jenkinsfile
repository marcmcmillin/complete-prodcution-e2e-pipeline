pipeline{
    agent{
        label "jenkins-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "complete-production-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "chaos662"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: "https://github.com/marcmcmillin/complete-prodcution-e2e-pipeline.git"
            }

        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }

        stage("Sonarqube Analysis"){
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                        sh 'mvn sonar:sonar'
                    }           
                }
            }
        }

        
        stage("Quality Gate"){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'       
                }
            }
        }
            
        
        stage("Build & Push Docker Image"){ 
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

       stage("Trivy Scan") {
            steps {
                script {
		            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image chaos662/complete-production-e2e-pipeline:1.0.0-119 --no-progress --scanners vuln --timeout 10m0s  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }

        stage("Trigger Continuous Pipeline"){ 
            steps {
                script {
                    sh "curl -v -k --user marc:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.mmchomelab.com/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"                              
                }
            }
        } 
    }
    post {
        always {
            script {
                def status = currentBuild.result ?: 'UNKNOWN'
                def color
                switch (status) {
                    case 'SUCCESS':
                        color = 'good'
                        break
                    case 'FAILURE':
                        color = 'danger'
                        break
                    default:
                        color = 'warning'
                }
                
                slackSend (channel: "#jenkins", message: "Update Deployment ${status.toLowerCase()} for ${env.JOB_NAME} ${env.BUILD_NUMBER} - ${env.BUILD_URL}", iconEmoji: ':jenkins:', color: color)
            }
        }
    }
}