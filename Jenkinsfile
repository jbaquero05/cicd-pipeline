pipeline {
    agent any

    tools {
        nodejs "node" 
    }

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'v1.0', description: 'Docker image tag to use')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo "Checking out branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Replace logo.svg (dev only) & Build Application') {
    steps {
        script {
            if (env.BRANCH_NAME == 'dev') {
                // Raw GitHub URL for direct download
                def logoUrl = "https://raw.githubusercontent.com/jbaquero05/image/7bc83b8a93d8c200c7162b90f8df0939e93f8b03/EPAM.svg"
                echo "Replacing logo.svg for branch ${env.BRANCH_NAME}"
                sh "rm -f src/logo.svg"
                sh "curl -sL '${logoUrl}' -o src/logo.svg"
            } else {
                echo "Preserving existing logo.svg for branch ${env.BRANCH_NAME}"
            }
        }

        echo 'Installing dependencies...'
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
                script {
                    def imageName = env.BRANCH_NAME == 'main' ? "nodemain" : "nodedev"
                    def fullTag = "${imageName}:${params.DOCKER_TAG ?: 'v1.0'}"
                    env.DOCKER_IMAGE = fullTag
                    echo "Building Docker image: ${fullTag}"
                    sh "docker build -t ${fullTag} ."
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    def containerName = env.BRANCH_NAME == 'main' ? "nodemain" : "nodedev"
                    def appPort = env.BRANCH_NAME == 'main' ? "3000" : "3001"

                    echo "Deploying ${env.DOCKER_IMAGE} to ${env.BRANCH_NAME} environment on port ${appPort}"

                    sh """
                        docker stop ${containerName} || true
                        docker rm ${containerName} || true
                    """

                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --name ${containerName} --expose 3000 -p 3000:3000 ${env.DOCKER_IMAGE}"
                    } else {
                        sh "docker run -d --name ${containerName} --expose 3001 -p 3001:3000 ${env.DOCKER_IMAGE}"
                    }

                    sleep(time: 10, unit: 'SECONDS')

                    sh "docker ps | grep ${containerName}"
                    echo "Node.js application successfully deployed at http://localhost:${appPort}"
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
