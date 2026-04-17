pipeline {
    agent any

    tools {
        jdk 'java17'
        maven 'maven'
    }

    environment {
        DOCKER_IMAGE = "rajeevgangaraju/poc-1:latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rajeevgangaraju/poc1.git'
            }
        }

        stage('Build & Unit Tests') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh '''
                dependency-check \
                  --scan . \
                  --format HTML \
                  --out dependency-check-report
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL $DOCKER_IMAGE'
            }
        }

        stage('Docker Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                sh '''
                docker rm -f poc-1 || true
                docker run -d \
                  -p 8081:8080 \
                  --name poc-1 \
                  $DOCKER_IMAGE
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check Jenkins logs."
        }
    }
}
