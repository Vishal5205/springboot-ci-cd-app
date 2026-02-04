pipeline {
    agent any
    tools { maven 'Maven3' }
    environment {
        DOCKER_IMAGE = 'vishal5205/springbootcicdapp'
    }
    stages {
        stage('Maven Build') { 
            steps { sh 'mvn clean package -DskipTests' } 
        }
        stage('SonarCloud Scan') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=vishal5205_springboot-ci-cd-app -Dsonar.organization=vishal5205 -Dsonar.host.url=https://sonarcloud.io'
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', 
                    passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_ID} ."
                    sh "docker tag ${DOCKER_IMAGE}:${BUILD_ID} ${DOCKER_IMAGE}:latest"
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_ID}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
    }
    post {
        success { 
            echo 'Pipeline complete. Docker Hub: https://hub.docker.com/r/vishal5205/springbootcicdapp'
        }
    }
}
