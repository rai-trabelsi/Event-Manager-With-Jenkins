pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        APP_NAME = 'events-app'
        IMAGE_NAME = 'ahmedrai/events-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        JAR_FILE = 'target/eventsProject-1.0.0.jar'
        GIT_REPO = 'https://github.com/ahmed-rai/event.git'
        GIT_BRANCH = 'ahmed'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Snyk SAST') {
            steps {
                snykSecurity(
                    snykInstallation: 'snyk',
                    snykTokenId: '305070a9-7c98-4731-b93f-af61bc8496ff',
                    failOnIssues: false,
                    additionalArguments: '--all-projects --detection-depth=5 --severity-threshold=medium'
                )
            }
        }

        stage('Trivy Git Secrets Scan') {
            steps {
                sh '''
                    trivy repo \
                      --scanners secret \
                      --format json \
                      -o trivy-git-repo-scan.json \
                      ${GIT_REPO}
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-git-repo-scan.json', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    test -f ${JAR_FILE}
                    cp ${JAR_FILE} .
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Run Docker Container') {
            steps {
                sh """
                    docker rm -f ${APP_NAME} || true
                    docker run -d --name ${APP_NAME} -p 8089:8089 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('ZAP DAST') {
            steps {
                sh '''
                    docker run --rm --network="host" zaproxy/zap-stable zap-baseline.py \
                      -t http://localhost:8089 \
                      -J zap-report.json || true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'zap-report.json', fingerprint: true, allowEmptyArchive: true
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh """
                    trivy image --format json -o trivy-image-scan.json ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-image-scan.json', fingerprint: true
                }
            }
        }

        stage('Snyk Container Scan') {
            steps {
                sh """
                    snyk container test ${IMAGE_NAME}:${IMAGE_TAG} --severity-threshold=medium || true
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
    }
}
