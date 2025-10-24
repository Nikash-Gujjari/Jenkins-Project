pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'github-jenkins-project-token' // Jenkins credential for pushing tags
        DOCKER_IMAGE = 'nikash-gujjari-python-app'              // Name of the Docker image
    }

    stages {
        stage('Nikash Gujjari - Build Docker Image') {
            steps {
                script{
                    // SCM 
                    checkout scm

                    // Fetch all tags
                    sh 'git fetch --tags'

                        // Get latest tag or default to v0
                    def latestTag = sh(script: "git describe --tags --abbrev=0 || echo v0", returnStdout: true).trim()

                    // Extract numeric part and increment
                    def versionNumber = latestTag.replaceAll("[^0-9]", "").toInteger()
                    def nextVersion = "v${versionNumber + 1}"
                    env.NEXT_VERSION = nextVersion

                    // Check if Dockerfile exists, if not, create this simple one
                    if (!fileExists('Dockerfile')) {
                        // Creates a Python slim Dockerfile
                        writeFile file: 'Dockerfile', text: '''
                        FROM python:3.12-slim
                        WORKDIR /app
                        COPY . .
                        RUN pip install --no-cache-dir -r requirements.txt || echo "No requirements.txt found"
                        CMD ["python", "nikashgujjari.py"]
                        '''.stripIndent()

                    }

                    sh "docker build -t ${env.DOCKER_IMAGE}:${env.NEXT_VERSION} ."
                }
            }
        }
        stage('Nikash Gujjari - Login to Dockerhub’ ') {
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''#!/bin/sh
                    docker login -u "$DOCKER_USER" -p "$DOCKER_PASS"
                    '''
                    }
                }
            }
        }
        stage('Nikash Gujjari - Push image to Dockerhub') {
            steps {
                script{
                    // Tag image for Docker Hub
                    def dockerHubRepo = 'ngujjari707/nikash-gujjari-python-app'
                    sh "docker tag ${env.DOCKER_IMAGE}:${env.NEXT_VERSION} ${dockerHubRepo}:${env.NEXT_VERSION}"

                    // Push to Docker Hub
                    sh "docker push ${dockerHubRepo}:${env.NEXT_VERSION}"
                }
            }
        }
    }
}