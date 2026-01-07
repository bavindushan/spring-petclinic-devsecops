pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')     // GitHub token credential ID
        SONARQUBE_AUTH = credentials('sonarqube-admin') // SonarQube admin credential ID
        IMAGE_NAME = "ghcr.io/bavindushan/spring-petclinic:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git(
                    url: 'https://github.com/bavindushan/spring-petclinic-devsecops.git',
                    credentialsId: 'github-token'
                )
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                sh './mvnw test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_HOST_URL = 'http://192.168.48.132:9000'
            }
            steps {
                sh """
                ./mvnw sonar:sonar \
                  -Dsonar.projectKey=spring-petclinic \
                  -Dsonar.host.url=$SONAR_HOST_URL \
                  -Dsonar.login=$SONARQUBE_AUTH
                """
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh """
                docker build -t $IMAGE_NAME .
                echo $GITHUB_TOKEN | docker login ghcr.io -u bavindushan --password-stdin
                docker push $IMAGE_NAME
                """
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh """
                trivy image --exit-code 1 $IMAGE_NAME
                """
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully!'
        }
        failure {
            echo 'CI pipeline failed!'
        }
    }
}

