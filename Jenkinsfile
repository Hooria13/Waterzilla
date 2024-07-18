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
                sh 'ls -la server'
            }
        }

       stage('Install Dependencies') {
            steps {
                dir('server') {
                    sh 'npm install'
                }
            }
        }

         stage('Create Source Code Archive') {
            steps {
                dir('server') {
                    // Zip the entire project directory
                    sh 'zip -r project-source.zip .'
                    archiveArtifacts artifacts: 'project-source.zip', allowEmptyArchive: true
                }
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

       // stage('Generate Report') {
         //   steps {
           //     sh 'your-command-to-generate-report'
             //   archiveArtifacts artifacts: '**/your-report*', allowEmptyArchive: true
            //}
        //}

        stage('Generate Report') {
    steps {
        // Assuming Trivy generates JSON report, adjust the command based on actual tool and format
        sh 'trivy --format json --output trivy-report.json ${DOCKER_IMAGE}:${env.BUILD_ID}'
        
        // Archive the vulnerability report as an artifact
        archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
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
