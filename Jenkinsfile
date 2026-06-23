pipeline {
    agent any

    options {
        ansiColor('xterm')                 // Colorized console output
        timeout(time: 20, unit: 'MINUTES') // Prevent jobs from hanging
    }

    environment {
        DEPLOY_HOST = "192.168.1.50"
        PLAYBOOK    = "tr/deploy.yml"      // playbook file in repo
    }

    stages {
        stage('Build') {
            steps {
                wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
                    bat 'mvn clean compile'
                }
            }
        }

        stage('Test') {
            steps {
                wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
                    bat 'mvn test'
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
                    bat 'mvn package'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            steps {
                wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
                    // Copy playbook from Jenkins workspace to VM
                    bat "scp ${PLAYBOOK} ubuntu@${DEPLOY_HOST}:/home/ubuntu/"
                    
                    // Run Ansible using the copied playbook
                    bat "ssh ubuntu@${DEPLOY_HOST} \"ansible-playbook /home/ubuntu/deploy.yml\""
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean workspace after build
        }
    }
}
