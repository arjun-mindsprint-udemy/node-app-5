pipeline {
    agent {
        label 'windows'
    }

    environment {
        DOCKERHUB_USERNAME = credentials('DOCKERHUB_USERNAME_ARJUN')
        DOCKERHUB_TOKEN = credentials('DOCKERHUB_TOKEN_ARJUN')
        COMMIT_ID = "${env.GIT_COMMIT.take(6)}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Set Commit ID') {
            steps {
                script {
                    env.COMMIT_ID = env.GIT_COMMIT.take(6)
                    echo "COMMIT_ID set to: ${env.COMMIT_ID}"
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                bat '''
                    echo %DOCKERHUB_TOKEN% | docker login --username %DOCKERHUB_USERNAME% --password-stdin
                '''
            }
        }

        // stage('Debug Directory Structure') {
        //     steps {
        //         bat '''
        //             echo "Root directory:"
        //             dir
        //             echo "src directory"
        //             dir src\\
        //             echo "springapi dir"
        //             dir src\\SpringAPI
        //             echo "target dir"
        //             dir src\\SpringAPI\\target
        //         '''
        //     }
        // }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def appName = 'node-app-5'
                    bat """
                        docker build -t arjun150800/${appName}:${env.COMMIT_ID} .
                        docker push arjun150800/${appName}:${env.COMMIT_ID}
                    """
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    def commitId = env.COMMIT_ID
                    bat """
                        wsl helm upgrade --install node-app-5 ./charts/node-app-5 -n dev --create-namespace --set image.tag=${commitId}
                    """
                }
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }
    }
}
