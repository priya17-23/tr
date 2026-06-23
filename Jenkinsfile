pipeline {
    agent any

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
                // Trigger Ansible on Linux VM
                bat 'ssh ubuntu@192.168.1.50 "ansible-playbook /home/ubuntu/deploy.yml -i /home/ubuntu/hosts.ini"'
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
