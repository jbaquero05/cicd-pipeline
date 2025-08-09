pipeline {
    agent any

    tools {
        nodejs "node"
    }
    
    environment {
        DOCKER_IMAGE = "${env.BRANCH_NAME == 'main' ? 'main:v1.0' : 'dev:v1.0'}"
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
    }
    
    stages {
        stage('Checkout SCM') {
            steps {
                echo "Checking out branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }
        
        stage('Replace logo.svg & Build Application') {
            steps {
                echo 'Replacing logo.svg'
                sh 'rm -f src/logo.svg'
                sh 'curl -sL "https://github.com/jbaquero05/image/blob/7bc83b8a93d8c200c7162b90f8df0939e93f8b03/EPAM.svg" -o src/logo.svg'
                echo 'Building the application...'
                sh 'npm install'
            }
        }
        
        stage('Test Application') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}"
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                echo "Deploying to ${env.BRANCH_NAME} environment on port ${APP_PORT}"
                script {
                    sh """
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    """
                    
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --name ${CONTAINER_NAME} --expose 3000 -p 3000:3000 ${DOCKER_IMAGE}"
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh "docker run -d --name ${CONTAINER_NAME} --expose 3001 -p 3001:3000 ${DOCKER_IMAGE}"
                    }

                    sleep(time: 10, unit: 'SECONDS')
                    
                    sh "docker ps | grep ${CONTAINER_NAME}"
                    echo "Nodejs application successfully deployed at http://localhost:${APP_PORT}"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            cleanWs()
        }
    }
}
