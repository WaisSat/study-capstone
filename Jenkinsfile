pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['LAB', 'QA', 'PROD'],
            description: 'Select environment to deploy'
        )
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        TARGET_ENV = "${params.ENVIRONMENT}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('CI - Ansible Syntax Check') {
            steps {
                // Ensure this path matches your repo structure
                sh 'ansible-playbook --syntax-check ansible/playbooks/deploy.yml'
            }
        }

        stage('CI - Dry Run') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'deploy-ssh',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    // Added --check for a true Dry Run
                    sh "ansible-playbook -i ansible/inventory/lab.ini ansible/playbooks/deploy.yml --check --private-key=${SSH_KEY}"
                }
            }
        }

        stage('Approval') {
            steps {
                input message: "Approve deployment to ${env.TARGET_ENV} environment?"
            }
        }

        stage('CD - Deploy') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'deploy-ssh',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh "ansible-playbook -i ansible/inventory/lab.ini ansible/playbooks/deploy.yml --private-key=${SSH_KEY}"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment to ${env.TARGET_ENV} completed successfully"
        }
        failure {
            echo "Pipeline failed for ${env.TARGET_ENV}"
        }
    }
}
