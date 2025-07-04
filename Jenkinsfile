pipeline {
    agent any

    environment {
        AZURE_WEBAPP_NAME     = "jenkins-tf-app-nitin"
        AZURE_RESOURCE_GROUP  = "jenkins-tf-rg"
        DEPLOY_PACKAGE        = "flask_app.zip"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/nitinpahwa77/Project-IAC-Uisng-Jenkins.git'
            }
        }

        stage('Terraform Init and Apply') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'azure-sp',
                    clientIdVariable: 'ARM_CLIENT_ID',
                    clientSecretVariable: 'ARM_CLIENT_SECRET',
                    tenantIdVariable: 'ARM_TENANT_ID',
                    subscriptionIdVariable: 'ARM_SUBSCRIPTION_ID'
                )]) {
                    dir('jenkins-tf-azure') {
                        sh '''
                            export ARM_CLIENT_ID=$ARM_CLIENT_ID
                            export ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET
                            export ARM_TENANT_ID=$ARM_TENANT_ID
                            export ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID

                            terraform init
                            terraform validate
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('flask-calculator') {
                    sh '''
                        apt-get update && apt-get install -y python3-venv

                        python3 -m venv venv
                        . venv/bin/activate
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Package App') {
            steps {
                dir('flask-calculator') {
                    sh 'zip -r ../${DEPLOY_PACKAGE} .'
                }
            }
        }

        stage('Deploy to Azure App Service') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'azure-sp',
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET',
                    tenantIdVariable: 'AZURE_TENANT_ID'
                )]) {
                    sh '''
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                        az webapp deploy --resource-group $AZURE_RESOURCE_GROUP --name $AZURE_WEBAPP_NAME --src-path ${DEPLOY_PACKAGE} --type zip
                    '''
                }
            }
        }
    }
}
