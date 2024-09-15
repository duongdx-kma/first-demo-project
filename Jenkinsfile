pipeline {
    agent none
    environment {
        NEXUS_URL = 'http://nexus-server:8081'
        SONAR_URL = 'http://sonarqube-server:9000'
        CODEDEPLOY_APPLICATION = 'my-app'
        CODEDEPLOY_DEPLOYMENT_GROUP = 'deploy-group'
    }
    stages {
        stage('Checkout') {
            agent { label 'maven-instance' }
            steps {
                checkout scm
            }
        }
        stage('Build') {
            agent { label 'maven-instance' }
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('UNIT TEST'){
            agent { label 'maven-instance' }
            steps {
                sh 'mvn test'
            }
        }
    }
    post {
        success {
            echo 'Build and deploy successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
