pipeline {
    agent any

    tools {
        jdk 'java-17'
        maven 'Maven3.9'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "prashikrk/loginwebappseven:latest"
    }

    stages {

        stage("Git Checkout") {
            steps {
                git branch: 'master',
                    url: 'https://github.com/prashikk71-ai/WebAppApplicationWith-Dockek.git'
            }
        }

        stage("Build & Test") {
            steps {
                sh "mvn clean verify"
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Loginwebapp \
                    -Dsonar.projectKey=Loginwebapp \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage("Build WAR") {
            steps {
                sh "mvn package"
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'prashikrk', toolName: 'docker') {
                        sh """
                        docker build -t $IMAGE_NAME .
                        docker push $IMAGE_NAME
                        """
                    }
                }
            }
        }

        stage("Trivy Image Scan") {
            steps {
                sh "trivy image $IMAGE_NAME > trivyimage.txt"
            }
        }

        stage("Deploy using Docker") {
            steps {
                sh """
                docker rm -f loginwebseven1 || true
                docker run -d \
                  --name loginwebseven1 \
                  -p 8083:8080 \
                  $IMAGE_NAME
                """
            }
        }
    }
}
