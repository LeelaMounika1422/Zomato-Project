pipeline {
    agent any

    environment {
        IMAGE_NAME = 'mounikagarapati001/zomato_new'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Project') {
            steps {
                echo 'Zomato project checkout completed'
                echo 'Preparing Jenkins CI/CD pipeline'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            "\${scannerHome}/bin/sonar-scanner" \
                            -Dsonar.projectKey=Zomato-Project \
                            -Dsonar.projectName=Zomato-Project \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    additionalArguments: '--scan . --format XML --disableYarnAudit --disableNodeAudit',
                    odcInstallation: 'DP-Check'
                )
            }
            post {
                always {
                    dependencyCheckPublisher(
                        pattern: '**/dependency-check-report.xml'
                    )
                }
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh 'trivy fs --scanners vuln --no-progress .'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --scanners vuln --no-progress ${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }

        stage('Docker Hub Push') {
            steps {
                withDockerRegistry(
                    credentialsId: 'docker-password',
                    url: 'https://index.docker.io/v1/'
                ) {
                    sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
                    sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest'
                    sh 'docker push ${IMAGE_NAME}:latest'
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f zomato-app || true
                    docker run -d \
                      --name zomato-app \
                      -p 3000:3000 \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo 'Zomato CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check console output.'
        }
    }
}