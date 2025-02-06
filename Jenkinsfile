pipeline {
    agent any

    triggers {
        githubPush() // Trigger the pipeline on GitHub push events
    }

    environment {
        NODE_HOME = 'C:\\Program Files\\nodejs' // Ensure correct Node.js installation path
        PATH = "${NODE_HOME};${env.PATH}" // Add Node.js to PATH

        AZURE_APP_NAME = 'jenkins-demo' // Replace with your Azure Web App name
        AZURE_RESOURCE_GROUP = 'SK' // Replace with your resource group
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SivaKrishna-2023/jenkins-demo.git', credentialsId: 'aef2135b-e5ad-4b3e-9bda-c07d57c16a85'
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    bat 'npm install'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    bat 'npm run build'
                }
            }
        }

        stage('Check and Delete Old ZIP') {
            steps {
                script {
                    bat '''
                    IF EXIST dist.zip (
                        DEL /F /Q dist.zip
                        echo Old dist.zip file deleted.
                    ) ELSE (
                        echo No old dist.zip file found.
                    )
                    '''
                }
            }
        }

        stage('Create ZIP File') {
            steps {
                script {
                    bat 'powershell -Command "Compress-Archive -Path dist\\* -DestinationPath dist.zip"'
                }
            }
        }

        stage('Approval Email Notification') {
            steps {
                script {
                    emailext (
                        subject: "Approval Required: Deploy to Azure - Build ${env.BUILD_NUMBER}",
                        body: """
                        <p>Hello Team,</p>
                        <p>The latest build <b>${env.BUILD_NUMBER}</b> is ready for deployment. Please approve or reject the deployment.</p>
                        <p><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        <p>Click the link above to approve or reject the deployment.</p>
                        <p>Best Regards,</p>
                        <p>Jenkins CI/CD</p>
                        """,
                        mimeType: 'text/html',
                        from: "eshwar.bashabathini88@mail.com",
                        to: "sivakrishna@middlewaretalents.com",
                        replyTo: "no-reply@middlewaretalents.com" // Optional: to set a reply-to address
                    )
                    input message: 'Do you approve the deployment?', ok: 'Deploy'
                }
            }
        }

        stage('Login to Azure') {
            steps {
                script {
                    withCredentials([ 
                        string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
                        string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'AZURE_CLIENT_SECRET'),
                        string(credentialsId: 'AZURE_TENANT_ID', variable: 'AZURE_TENANT_ID') 
                    ]) {
                        bat '''
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        '''
                    }
                }
            }
        }

        stage('Deploy to Azure') {
            steps {
                script {
                    bat '''
                    az webapp deployment source config-zip --resource-group SK --name jenkins-demo --src dist.zip
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Deployment process finished.'
        }
    }
}
