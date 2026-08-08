pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                sh 'echo "Checkout successful"'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                sh 'echo "Static site ready for packaging"'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('sonarqube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=shapel-website-devops \
                            -Dsonar.sources=src/
                        """
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "shaunjohnmathew/static-website:${BUILD_NUMBER}"
            }

            steps {
                script {

                    sh 'docker build -t ${DOCKER_IMAGE} .'

                    def dockerImage = docker.image("${DOCKER_IMAGE}")

                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        'docker-cred'
                    ) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "shapel-website-devops"
                GIT_USER_NAME = "shaunjohn-04"
            }

            steps {
                withCredentials([
                    string(
                        credentialsId: 'github',
                        variable: 'GITHUB_TOKEN'
                    )
                ]) {

                    sh '''
                        git config user.email "shaunjohnmathew04@gmail.com"
                        git config user.name "${GIT_USER_NAME}"

                        sed -i "s|image: .*|image: shaunjohnmathew/static-website:${BUILD_NUMBER}|g" k8s/deployment.yml

                        git add k8s/deployment.yml

                        git commit -m "Update static site image tag to ${BUILD_NUMBER} [skip ci]" || echo "No changes to commit"

                        git push https://${GIT_USER_NAME}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }
    }
}
