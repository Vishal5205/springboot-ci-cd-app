pipeline {
    agent any
    tools { 
        maven 'Maven3' 
    }
    environment {
        SONAR_PROJECT = 'vishal5205_springboot-ci-cd-app'
        DOCKER_IMAGE = 'vishal5205/springbootcicdapp'
    }
    stages {
        stage('Checkout') { 
            steps { checkout scm } 
        }
        stage('Maven Build') { 
            steps { sh 'mvn clean package -DskipTests' } 
        }
        stage('SonarCloud Scan') {
            steps {
                withSonarQubeEnv('SonarCloud') {  // ← Match EXACT Jenkins config name
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT} \
                        -Dsonar.organization=vishal5205 \
                        -Dsonar.host.url=https://sonarcloud.io
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_ID} ."
                    sh "docker tag ${DOCKER_IMAGE}:${BUILD_ID} ${DOCKER_IMAGE}:latest"
                    echo "Docker images built: ${BUILD_ID} and latest"
                }
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', 
                    passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_ID}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                    echo "Docker images pushed successfully"
                }
            }
        }
    }
    post {
        success { 
            echo 'Pipeline completed successfully - check Docker Hub and SonarCloud!'
        }
        failure { 
            echo 'Pipeline failed - check logs' 
        }
    }
}
