pipeline {
    agent any

    tools {
        maven 'Maven-3'
    }

    environment {
        DOCKERHUB = credentials('dockerhub')
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_URL = "http://<your-sonar-ip>:9000"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/your/repo.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarServer') {
                    sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=simple-java-app \
                      -Dsonar.host.url=${SONAR_URL} \
                      -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t dockerhubusername/simple-java-app:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                echo ${DOCKERHUB_PSW} | docker login -u ${DOCKERHUB_USR} --password-stdin
                docker push dockerhubusername/simple-java-app:${BUILD_NUMBER}
                """
            }
        }
    }
}

