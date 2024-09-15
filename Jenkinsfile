pipeline {
    agent none
    environment {
        NEXUS_URL = 'http://nexus-server:8081'
        SONAR_URL = 'http://sonarqube-server:9000'
        CODEDEPLOY_APPLICATION = 'my-app'
        CODEDEPLOY_DEPLOYMENT_GROUP = 'deploy-group'
        TOMCAT_USER = 'ec2-user'  // Adjust as per your setup
        TOMCAT_IP = '10.100.4.66'
        TOMCAT_PORT = '8080'
        TOMCAT_WEBAPPS_DIR = '/opt/tomcat/latest/webapps'
        SSH_KEY_ID = 'tomcat-ssh-key'  // Jenkins SSH private key credential ID
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

        stage('Deploy to Tomcat') {
            agent { label 'master' }
            steps {
                withCredentials([file(credentialsId: "${SSH_KEY_ID}", variable: 'SSH_KEY')]) {
                    script {
                        // Clean up Tomcat webapps directory (remove all except ROOT)
                        sh """
                            ssh -i ${SSH_KEY} ${TOMCAT_USER}@${TOMCAT_IP} \\
                            'sudo rm -rf ${TOMCAT_WEBAPPS_DIR}/{docs,examples,host-manager,manager,ROOT}/*'
                        """

                        // Deploy the WAR file as ROOT.jar
                        sh """
                            scp -i ${SSH_KEY} target/*.jar ${TOMCAT_USER}@${TOMCAT_IP}:${TOMCAT_WEBAPPS_DIR}/ROOT.jar
                        """

                        // Optionally restart Tomcat
                        sh """
                            ssh -i ${SSH_KEY} ${TOMCAT_USER}@${TOMCAT_IP} \\
                            'sudo systemctl restart tomcat'
                        """
                    }
                }
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
