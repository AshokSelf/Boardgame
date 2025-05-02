pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'outlookdock/boardgame'
        DOCKER_TAG = 'tagname'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AshokSelf/Boardgame.git'
                echo "Repository checked out successfully"
            }
        }
        stage('Trivy file scan') {
            steps {
                sh 'trivy fs .'
            }
        }
        stage('Code Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Code Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File System Scan') {
            steps {
                sh 'wget https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O trivy-html.tpl'
                sh 'trivy fs --format template --template @trivy-html.tpl -o trivy-fs-report.html .'
                archiveArtifacts artifacts: 'trivy-fs-report.html', fingerprint: true
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        def scannerHome = tool 'SonarQubeScanner'
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectName=BoardGame \
                            -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=.
                        """
                    }
                }
            }
        }
        stage('Quality Gates') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-tocken'
                }
            }
        }
        stage('Build Package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Set Version') {
            steps {
                script {
                    sh "mvn versions:set -DnewVersion=0.0.${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    retry(3) {
                        docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    }
                }
            }
        }
        stage('Tag Docker Image') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").tag('latest')
                }
            }
        }
        stage('Trivy Docker Image Scan') {
            steps {
                sh 'wget https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O trivy-html.tpl'
                sh "trivy image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                trivy image --format template --template "@/contrib/html.tpl" -o trivy-image-report.html ("${DOCKER_IMAGE}:${DOCKER_TAG}")

        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
