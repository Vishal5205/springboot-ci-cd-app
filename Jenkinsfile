pipeline {
    agent any
    tools { maven 'Maven3' }
    environment {
        SONAR_PROJECT = 'Vishal5205_springboot-ci-cd-app'
        DOCKER_IMAGE = 'vishal5205/springbootcicdapp'
    }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Maven Build') {
            steps { sh 'mvn clean package -DskipTests' }
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
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    img = docker.build("${DOCKER_IMAGE}:${BUILD_ID}")
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        img.push()
                        img.push('latest')
                    }
                }
            }
        }
    }
}
