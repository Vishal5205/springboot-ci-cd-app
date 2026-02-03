pipeline {
    agent any
    tools { maven 'Maven3' }
    environment {
        SONAR_PROJECT = 'Vishal5205_springboot-ci-cd-app'
        DOCKER_IMAGE = 'vishal5205/springbootcicdapp'
    }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Maven Build') { steps { sh 'mvn clean package -DskipTests' } }
        stage('SonarCloud Scan') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=$SONAR_PROJECT -Dsonar.organization=vishal5205'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    def img = docker.build("${DOCKER_IMAGE}:${BUILD_ID}")
                    echo "Docker image built: ${img.id}"
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        docker.image("${DOCKER_IMAGE}:${BUILD_ID}").push('latest')
                    }
                    echo "Docker image pushed to Docker Hub"
                }
            }
        }
    }
    post {
        success { echo 'Pipeline completed successfully' }
        failure { echo 'Pipeline failed - check logs' }
    }
}
