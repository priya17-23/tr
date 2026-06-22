pipeline {
    agent any

    options {
        timeout(time: 20, unit: 'MINUTES') // prevent jobs from hanging
    }

    environment {
        DEPLOY_HOST = "192.168.1.50"
        PLAYBOOK    = "TR\\deploy.yml"   // Windows path to playbook in repo
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
                // Show workspace contents to confirm file paths
                bat "dir TR"

                // Copy only deploy.yml from workspace to VM
                bat "scp TR\\deploy.yml ubuntu@${DEPLOY_HOST}:/home/ubuntu/deploy.yml"

                // Run Ansible using the copied playbook
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
