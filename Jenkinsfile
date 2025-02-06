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

        stage('Approval Required') {
            steps {
                script {
                    emailext subject: "🔔 Approval Needed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                             body: """
                             <p>🚀 Build <b>${env.BUILD_NUMBER}</b> for <b>${env.JOB_NAME}</b> is awaiting approval.</p>
                             <p>Please review and approve the build:</p>
                             <p><a href="${env.BUILD_URL}input">🔗 Approve Here</a></p>
                             <p>Details: <a href="${env.BUILD_URL}">Jenkins Build URL</a></p>
                             """,
                             to: 'Sivakrishna@middlewaretalents.com',
                             from: 'eshwar.bashabathini88@mail.com',
                             mimeType: 'text/html'
                }
                timeout(time: 30, unit: 'MINUTES') {  // Approval must be given within 30 minutes
                    input message: "Do you approve the build?", ok: "Approve"
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
