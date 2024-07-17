pipeline {
    agent any

    //environment {
        // Define environment variables as needed
     //   DOCKER_REGISTRY = 'your-docker-registry'  // Replace with your Docker registry URL
       // IMAGE_NAME = 'your-image-name'
        //VERSION = "1.0.${env.BUILD_NUMBER}"  // Versioning using Jenkins build number
    //}

    stages {
        stage('Clone') {
            steps {
                // Checkout code from GitHub repository
                git 'https://github.com/Hooria13/Waterzilla.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Example: Install dependencies (adjust based on your project)
                sh 'npm install'  // Replace with your dependency installation command
            }
        }

        stage('Build Artifacts') {
            steps {
                // Example: Build artifacts (adjust based on your project)
                sh 'npm run build'  // Replace with your build command
            }
            post {
                success {
                    // Archive artifacts (adjust based on your project)
                    archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Create Dockerfile (replace contents with your Dockerfile)
                    writeFile file: 'Dockerfile', text: '''
                        FROM node:alpine
                        WORKDIR /app
                        COPY . .
                        RUN npm install --production
                        CMD ["node", "index.js"]
                    '''
                }
                // Build Docker image
                docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}")
            }
            post {
                success {
                    // Scan Docker image for vulnerabilities (adjust based on your tool)
                    sh 'docker scan "${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}" --file Dockerfile --severity High'
                    // Push Docker image to registry
                    docker.withRegistry('https://${DOCKER_REGISTRY}', 'docker-hub-credentials') {
                        dockerImage.push("${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}")
                    }
                }
            }
        }

        stage('Generate Vulnerability Report') {
            steps {
                // Example: Generate vulnerability report (adjust based on your tool)
                sh 'docker scan "${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}" --file Dockerfile --json > vulnerability_report.json'
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                // Example: Create Docker Compose file (adjust based on your project)
                writeFile file: 'docker-compose.yml', text: '''
                    version: '3'
                    services:
                      app:
                        image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION}
                        ports:
                          - "8080:8080"
                '''
                // Deploy with Docker Compose
                sh 'docker-compose up -d'
            }
        }
    }

    post {
        always {
            // Cleanup steps if necessary
            deleteDir()
        }
    }
}
