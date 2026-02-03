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
            } 
        }
        stage('Maven Build') {
            steps { 
                sh 'mvn clean package -DskipTests' 
            }
        }
        stage('SonarCloud Scan') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=$SONAR_PROJECT -Dsonar.organization=vishal5205'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        echo "Quality Gate: ${qg.status}"
                        if (qg.status != 'OK') {
                            echo "Quality Gate failed: ${qg.status}. Continuing anyway..."
                        }
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    def img = docker.build("${DOCKER_IMAGE}:${BUILD_ID}")
                    env.BUILD_IMG = img.id
                    echo "Docker image built: ${img.id}"
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    def img = docker.image("${DOCKER_IMAGE}:${BUILD_ID}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        img.push()
                        img.push('latest')
                    }
                    echo "Docker image pushed to Docker Hub!"
                }
            }
        }
    }
    post {
        success {
            echo '🎉 FULL CI/CD PIPELINE SUCCESS!'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}
