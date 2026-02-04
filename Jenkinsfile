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
        stage('Docker Build') {
            steps { 
                sh "sudo docker build -t ${DOCKER_IMAGE}:${BUILD_ID} ."
                sh "sudo docker tag ${DOCKER_IMAGE}:${BUILD_ID} ${DOCKER_IMAGE}:latest"
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', 
                    passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'echo $PASS | sudo docker login -u $USER --password-stdin'
                    sh "sudo docker push ${DOCKER_IMAGE}:${BUILD_ID}"
                    sh "sudo docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
    }
}
