pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Pull the latest code from the corresponding branch
                    checkout scm
                }
            }
        }

        stage('Build Image on Jenkins') {
            when {
                anyOf {
                    branch 'dev'
                    branch 'testing'
                    branch 'main'
                }
            }
            steps {
                script {
                    echo "Building Docker image for branch: ${env.BRANCH_NAME}"
                    
                    sh """
                      docker build -t my-node-app:${env.BRANCH_NAME} .
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            when {
                branch 'dev'
            }
            steps {
                script {
                    echo "Deploying to Dev environment..."
                    
                    sshagent (credentials: ['DEV_SSH_KEY']) {
                        sh """
                          ssh -o StrictHostKeyChecking=no ubuntu@<DEV_SERVER_IP> \\
                          'docker stop app_dev || true && docker rm app_dev || true && \\
                           docker run -d --name app_dev -p 3000:3000 my-node-app:dev'
                        """
                    }
                }
            }
        }

        stage('Deploy to Test') {
            when {
                branch 'testing'
            }
            steps {
                script {
                    echo "Deploying to Test environment..."
                    
                    sshagent (credentials: ['DEV_SSH_KEY']) {
                        sh """
                          ssh -o StrictHostKeyChecking=no ubuntu@<TEST_SERVER_IP> \\
                          'docker stop app_testing || true && docker rm app_testing || true && \\
                           docker run -d --name app_testing -p 3000:3000 my-node-app:testing'
                        """
                    }
                }
            }
        }

        stage('Run Tests') {
            when {
                branch 'testing'
            }
            steps {
                script {
                    echo "Running automated tests on the Test environment..."
                    
                    sshagent (credentials: ['DEV_SSH_KEY']) {
                        sh """
                          ssh -o StrictHostKeyChecking=no ubuntu@<TEST_SERVER_IP> \\
                          'docker exec app_testing npm test'
                        """
                    }
                }
            }
        }

        stage('Merge Testing to Main') {
            when {
                allOf {
                    branch 'testing'
                    expression { currentBuild.currentResult == "SUCCESS" }
                }
            }
            steps {
                script {
                    echo "All tests passed. Merging from testing to main..."
                    
                    sh """
                      git config user.email "jenkins@example.com"
                      git config user.name "Jenkins CI"
                      git checkout testing
                      git pull origin testing
                      git checkout main
                      git pull origin main
                      git merge testing
                      git push origin main
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo "Deploying to Staging environment..."
                    
                    sshagent (credentials: ['DEV_SSH_KEY']) {
                        sh """
                          ssh -o StrictHostKeyChecking=no ubuntu@<STAGING_SERVER_IP> \\
                          'docker stop app_staging || true && docker rm app_staging || true && \\
                           docker run -d --name app_staging -p 3000:3000 my-node-app:main'
                        """
                    }
                }
            }
        }
    }
}
