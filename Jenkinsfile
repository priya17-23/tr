pipeline {
    agent any

    options {
        ansiColor('xterm')                 // Colorized console output
        timeout(time: 20, unit: 'MINUTES') // Prevent jobs from hanging
    }

    environment {
        DEPLOY_HOST = "192.168.1.50"
        INVENTORY   = "tr/hosts.ini"   // inventory file in repo
        PLAYBOOK    = "tr/deploy.yml"  // playbook file in repo
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
                    // Copy playbook + inventory from Jenkins workspace to VM
                    bat "scp ${PLAYBOOK} ${INVENTORY} ubuntu@${DEPLOY_HOST}:/home/ubuntu/"
                    
                    // Run Ansible using the copied files
                    bat "ssh ubuntu@${DEPLOY_HOST} \"ansible-playbook /home/ubuntu/deploy.yml -i /home/ubuntu/hosts.ini\""
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
