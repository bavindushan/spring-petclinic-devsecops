pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')
        IMAGE_NAME = "ghcr.io/bavindushan/spring-petclinic:latest"
    }

    stages {

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
                sh 'trivy image --exit-code 1 $IMAGE_NAME'
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

