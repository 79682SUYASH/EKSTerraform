pipeline {
    agent any

    environment {
        AWS_CREDENTIALS_ID = "aws-creds"
        WORKSPACE_BIN = "${WORKSPACE}/bin"
        PATH = "${WORKSPACE_BIN}:$PATH"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/79682SUYASH/EKSTerraform.git'
            }
        }

        stage('Install Terraform') {
            steps {
                sh '''
                  mkdir -p $WORKSPACE/bin
                  if ! command -v terraform >/dev/null 2>&1; then
                    curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor > $WORKSPACE/hashicorp-keyring.gpg
                    echo "deb [signed-by=$WORKSPACE/hashicorp-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" > $WORKSPACE/hashicorp.list
                    sudo mv $WORKSPACE/hashicorp.list /etc/apt/sources.list.d/
                    sudo mv $WORKSPACE/hashicorp-keyring.gpg /usr/share/keyrings/
                    sudo apt update -y
                    sudo apt install -y terraform
                  fi
                  terraform -version
                '''
            }
        }

        stage('Terraform Init') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: 'us-east-1') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: 'us-east-1') {
                    sh 'terraform plan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: 'us-east-1') {
                    sh 'terraform apply -auto-approve'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Terraform EKS Cluster creation completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
