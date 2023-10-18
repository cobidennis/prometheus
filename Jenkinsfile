pipeline {
    agent any
    
    stages {
        stage('Download Code from GHub') {
            steps {
                sh '''
                    mkdir workspace
                    cd workspace
                    git clone https://github.com/cobidennis/hrapp-nodes-config.git
                '''

            }
        }
        stage('Download Env File from S3') {
            steps {
                script {
                    def s3Bucket = 'dobee-buckets'
                    def keyFile = 'keys/DobeeP53.pem'
                    def envFilePath = 'workspace'

                    sh "aws s3 cp s3://${s3Bucket}/${keyFile} ${envFilePath}"

                    keyFileExists = fileExists("${envFilePath}/DobeeP53.pem")

                    if (keyFileExists) {
                        echo "AWS Key File Exists. Proceeding to the next stage."
                    } else {
                        error "AWS Key File Not Found. Aborting the pipeline."
                    }
                }
            }
        }
        stage ('Ansible: Deploy Apps and Monitoring System') {
            when {
                expression {
                    return keyFileExists
                }
            }
            steps {
                sh '''
                    cd workspace/hrapp-nodes-config/ansible
                    ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook -i inventory.ini  playbook-monitoring.yml --key-file ../../DobeeP53.pem -u ec2-user
                '''
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}