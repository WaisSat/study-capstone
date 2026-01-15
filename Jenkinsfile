pipeline { 
    agent any 
 
    environment { 
        ANSIBLE_HOST_KEY_CHECKING = 'False' 
    } 
 
    stages { 
 
        stage('Checkout') { 
            steps { 
                checkout scm 
            } 
        } 
 
        stage('CI - Ansible Syntax Check') { 
            steps { 
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
                    sh ''' 
                    ansible-playbook \ 
                      -i ansible/inventory/lab.ini \ 
                      ansible/playbooks/deploy.yml \ 
                    ''' 
                } 
            } 
        } 
 
        stage('Approval') { 
            steps { 
                input message: 'Approve deployment to LAB environment' 
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
                    sh ''' 
                    ansible-playbook \ 
                      -i ansible/inventory/lab.ini \ 
                      ansible/playbooks/deploy.yml 
                    ''' 
                } 
            } 
        } 
    } 
 
    post { 
        success { 
            echo 'Deployment completed successfully' 
        } 
        failure { 
            echo 'Pipeline failed' 
        } 
    } 
} 

