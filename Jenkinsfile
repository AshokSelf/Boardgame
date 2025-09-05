pipeline {
    agent any
    tools {
        jdk 'jdk-21'
        maven 'maven3'
    }
    stages {
        stage('Git-Clone') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: 'https://github.com/AshokSelf/java-project.git'
            }
        }
        stage('Maven-Build-and-Test') {
            steps {
                echo 'Building and testing with Maven...'
                sh 'mvn clean'
                sh 'mvn compile'
                sh 'mvn test'
                
            }
        }
    }
    post {
        always {
            echo 'This will always run after the stages have completed.'
            cleanWs()
        }
        success {
            echo 'The pipeline completed successfully! Your application is ready.'
        }
        failure {
            echo 'The pipeline failed. Please check the logs for details.'
        }
    }
}
