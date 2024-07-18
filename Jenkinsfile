pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "Hooria13/nodejs-project"
        DOCKER_REGISTRY_CREDENTIALS_ID = 'docker-cred'
        GIT_REPO = 'https://github.com/Hooria13/Waterzilla.git'
        GIT_CREDENTIALS_ID = 'Git-Credentials13'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', credentialsId: "${GIT_CREDENTIALS_ID}", url: "${GIT_REPO}"
                sh 'ls -la'
                sh 'ls -la Hooria13'
                sh 'ls -la Hooria13/Waterzilla'
                sh 'ls -la Hooria13/Waterzilla/server'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('Hooria13/Waterzilla/server') {
                    sh 'npm install'
                }
            }
        }

        stage('Build Artifacts') {
            steps {
                sh 'your-command-to-build-artifacts'
                archiveArtifacts artifacts: '**/your-artifacts*', allowEmptyArchive: true
            }
        }

      
        stage('Create Dockerfile') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: """
                    FROM node:14-alpine
                    WORKDIR /app
                    COPY package.json ./
                    COPY package-lock.json ./
                    RUN npm install
                    COPY . .
                    CMD ["node", "index.js"]
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }

       
        stage('Scan Docker Image') {
            steps {
                script {
                    sh "trivy image ${DOCKER_IMAGE}:${env.BUILD_ID}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_REGISTRY_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Generate Report') {
            steps {
                sh 'your-command-to-generate-report'
                archiveArtifacts artifacts: '**/your-report*', allowEmptyArchive: true
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    writeFile file: 'docker-compose.yml', text: """
                    version: '3.8'
                    services:
                      app:
                        image: ${DOCKER_IMAGE}:${env.BUILD_ID}
                        ports:
                          - "8080:8080"
                    """
                    }
                    sh 'docker-compose up -d'
                }
            }
        }
    
    post {
        always {
            cleanWs()
        }
    }
}
