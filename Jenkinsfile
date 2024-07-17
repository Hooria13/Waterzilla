pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'Alpine'
        VERSION = sh(returnStdout: true, script: 'echo $BUILD_NUMBER').trim()
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Hooria13/Waterzilla.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'  // Adjust as per your project's dependency management
            }
        }

        stage('Build Artifacts') {
            steps {
                sh 'npm run build'  // Adjust as per your project's build process
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/build/**', allowEmptyArchive: true
                }
            }
        }

        stage('Create Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}", "-f Dockerfile.alpine .")
                    docker.withRegistry('', 'docker-credentials13') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Scan Docker Image') {
            steps {
                script {
                    def report = docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}").securityChecker()
                    writeFile file: 'docker-image-report.txt', text: report
                    echo 'Security vulnerabilities report:'
                    echo report
                }
            }
        }

        stage('Upload Docker Image') {
            steps {
                script {
                    docker.withRegistry("${DOCKER_REGISTRY}", 'docker-credentials13') {
                        dockerImage.push("${VERSION}")
                    }
                }
            }
        }

        stage('Create Docker Compose File and Deploy') {
            steps {
                script {
                    writeFile file: 'docker-compose.yml', text: """
                        version: '3'
                        services:
                            ${IMAGE_NAME}:
                                image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}
                                ports:
                                    - '80:8080'  # Adjust port mapping as per your application needs
                    """
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}
