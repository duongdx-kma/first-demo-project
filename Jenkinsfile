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
                    archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
                    stash name: 'app-war', includes: '**/target/*.war' // save the artifact for Deploy State
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
            agent { label 'built-in' }
            steps {
                unstash 'app-war' // get the artifact from Buil stage
                withCredentials([sshUserPrivateKey(credentialsId: "${SSH_KEY_ID}", keyFileVariable: 'SSH_KEY_FILE')]) {
                    script {
                        // Clean up Tomcat webapps directory (remove all except ROOT)
                        sh """
                            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_FILE} ${TOMCAT_USER}@${TOMCAT_IP} \\
                            'sudo rm -rf ${TOMCAT_WEBAPPS_DIR}/{docs,examples,host-manager,manager,ROOT}'
                        """

                        // Deploy the WAR file to  /home/ec2-user/ROOT.war
                        sh """
                            scp -o StrictHostKeyChecking=no -i ${SSH_KEY_FILE} target/*.war ${TOMCAT_USER}@${TOMCAT_IP}:/home/$TOMCAT_USER/ROOT.war
                        """

                        // Deploy the WAR file to /opt/tomcat/latest/webapps/ROOT.war
                        sh """
                            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_FILE} ${TOMCAT_USER}@${TOMCAT_IP} \\
                            'sudo cp /home/$TOMCAT_USER/ROOT.war $TOMCAT_WEBAPPS_DIR/ROOT.war'
                        """

                        // Optionally restart Tomcat
                        sh """
                            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_FILE} ${TOMCAT_USER}@${TOMCAT_IP} \\
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
