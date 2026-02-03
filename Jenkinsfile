pipeline {
    agent any
    tools { maven 'Maven3' }
    environment {
        SONAR_PROJECT = 'Vishal5205_springboot-ci-cd-app'
        DOCKER_IMAGE = 'vishal5205/springbootcicdapp'
    }
    stages {
        stage('Checkout') { 
            steps { 
                checkout scm 
                echo "Checked out from GitHub"
            } 
        }
        stage('Maven Build') {
            steps { 
                sh 'mvn clean package -DskipTests'
                echo "Maven build SUCCESS - JAR created"
            }
        }
        stage('SonarCloud Scan') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=$SONAR_PROJECT -Dsonar.organization=vishal5205'
                }
                echo "SonarCloud analysis completed"
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    def qg = waitForQualityGate()
                    echo "Quality Gate Status: ${qg.status}"
                    if (qg.status == 'ERROR') {
                        error "Quality gate failed: ${qg.status}"
                    }
                    echo "Quality Gate PASSED (${qg.status}) - Continue to Docker"
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    def image = docker.build("${DOCKER_IMAGE}:${BUILD_ID}")
                    env.DOCKER_IMAGE_ID = image.id
                    echo "Docker image built: ${image.id}"
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    def image = docker.image("${DOCKER_IMAGE}:${BUILD_ID}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        image.push("${BUILD_ID}")
                        image.push('latest')
                    }
                    echo " Docker image pushed: vishal5205/springbootcicdapp:latest"
                }
            }
        }
    }
    post {
        success {
            echo 'FULL PIPELINE SUCCESS! Ready for ArgoCD deployment'
        }
        failure {
            echo ' Pipeline failed - check stage logs above'
        }
    }
}
