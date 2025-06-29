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
                git 'https://github.com/nitinpahwa77/Project-IAC-Uisng-Jenkins.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('flask-calculator') {
                    sh '''
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
