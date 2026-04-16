pipeline {
    agent any
    environment {
        TF_VAR_gcp_project = "qwiklabs-gcp-02-1264a5a6c0ad"
        TF_VAR_bucket = "tf-remote-state-student_03_d14a424e7b9f-15621-16238"
        REPOSITORY = "https://github.com/andymooreno4-star/sample-awx-project"
        TF_VAR_pubkey_path = "${WORKSPACE}/ansible_key.pub"
    }
    stages {
        stage('Generate public key') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "SSH_KEY", keyFileVariable: "SSH_KEY")]) {
                    sh "ssh-keygen -f $SSH_KEY -y > ${WORKSPACE}/ansible_key.pub"
                }
            }
        }
        stage('Terraform Init') {
            steps {
                dir("terraform") {
                    withCredentials([file(credentialsId: 'gcp-svc-acct', variable: 'GOOGLE_CLOUD_KEYFILE_JSON')]) {
                        sh "ansible 127.0.0.1 -m template -a 'src=${WORKSPACE}/terraform/main.tftemp dest=${WORKSPACE}/terraform/main.tf' -e 'bucket=$TF_VAR_bucket'"
                        sh '''
                        export GOOGLE_APPLICATION_CREDENTIALS=$GOOGLE_CLOUD_KEYFILE_JSON
                        terraform init
                        '''
                    }
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                dir("terraform") {
                    withCredentials([file(credentialsId: 'gcp-svc-acct', variable: 'GOOGLE_CLOUD_KEYFILE_JSON')]) {
                        sh '''
                        export GOOGLE_APPLICATION_CREDENTIALS=$GOOGLE_CLOUD_KEYFILE_JSON
                        terraform plan
                        '''
                    }
                }
            }
        }
        stage('Terraform Apply') {
            steps {
                dir("terraform") {
                    withCredentials([file(credentialsId: 'gcp-svc-acct', variable: 'GOOGLE_CLOUD_KEYFILE_JSON')]) {
                        sh '''
                        export GOOGLE_APPLICATION_CREDENTIALS=$GOOGLE_CLOUD_KEYFILE_JSON
                        terraform apply -auto-approve
                        '''
                    }
                }
            }
        }
        stage('Ansible Playbook') {
            steps {
                dir("ansible") {
                    withCredentials([sshUserPrivateKey(credentialsId: "SSH_KEY", keyFileVariable: "ANSIBLE_PRIVATE_KEY_FILE"), file(credentialsId: 'gcp-svc-acct', variable: 'GOOGLE_CLOUD_KEYFILE_JSON')]) {
                        sh "ansible 127.0.0.1 -m template -a 'src=${WORKSPACE}/ansible/inventory_template.yml dest=${WORKSPACE}/ansible/inventory.gcp_compute.yml' -e 'project_id=${TF_VAR_gcp_project}'" 
                        sh '''
                        export GOOGLE_APPLICATION_CREDENTIALS=$GOOGLE_CLOUD_KEYFILE_JSON
                        ansible-playbook -i inventory.gcp_compute.yml playbook.yml -e "repository_url=${REPOSITORY}"
                        '''
                    }
                }
            }
        }
    }
}
