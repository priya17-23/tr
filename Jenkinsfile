pipeline {
    agent any

    options {
        ansiColor('xterm')   // Colorized console output
        timeout(time: 20, unit: 'MINUTES')
    }

    environment {
        DEPLOY_HOST = "192.168.1.50"
        INVENTORY   = "TR/hosts.ini"   // keep inventory in repo too
        PLAYBOOK    = "TR/deploy.yml"  // playbook path in repo
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Package') {
            steps {
                bat 'mvn package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                // Copy playbook + inventory from Jenkins workspace to VM
                bat "scp ${PLAYBOOK} ${INVENTORY} ubuntu@${DEPLOY_HOST}:/home/ubuntu/"

                // Run Ansible using the copied files
                bat "ssh ubuntu@${DEPLOY_HOST} \"ansible-playbook /home/ubuntu/deploy.yml -i /home/ubuntu/hosts.ini\""
            }
        }
    }

    post {
        success {
            echo '✅ Build and Deploy Successful'
        }
        failure {
            echo '❌ Build or Deploy Failed'
        }
    }
}
