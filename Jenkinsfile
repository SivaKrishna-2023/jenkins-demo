pipeline {

    agent any
    
    triggers {

     githubPush() // Trigger the pipeline on GitHub push events

  }
 

    environment {
        // Define the Node.js home directory
        NODE_HOME = 'C:\\Program Files\\nodejs' // Update this path based on your Node.js installation
        PATH = "${NODE_HOME};${env.PATH}"      // Add Node.js to the PATH and retain existing PATH

        AZURE_APP_NAME = 'jenkins-demo' // Replace with your Azure Web App name
        AZURE_RESOURCE_GROUP = 'SK' // Replace with your resource group
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout the code from your Git repository and specify the branch
                git branch: 'main', url: 'https://github.com/SivaKrishna-2023/jenkins-demo.git', credentialsId : 'aef2135b-e5ad-4b3e-9bda-c07d57c16a85'
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    // Install Node.js dependencies (Windows command)
                    bat 'npm install'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Build the frontend application (Windows command)
                    bat 'npm run build'
                }
            }
        }

        // Stage to check if the ZIP file exists and delete it if present
        stage('Check and Delete Old ZIP') {
            steps {
                script {
                    // Check if the dist.zip file exists and delete it if present
                    bat '''
                    IF EXIST dist.zip (
                        DEL /F /Q dist.zip
                        echo "Old dist.zip file deleted."
                    ) ELSE (
                        echo "No old dist.zip file found."
                    )
                    '''
                }
            }
        }

        stage('Create ZIP File') {
            steps {
                script {
                    // Create a zip file of the build output (assuming build output is in "dist" folder)
                    bat 'powershell -Command "Compress-Archive -Path dist\\* -DestinationPath dist.zip"'
                }
            }
        }

         stage('Login to Azure') {
            steps {
                script {
                    withCredentials([  // Access Azure secrets from GitHub's secret manager (Jenkins Credentials)
                        string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'), 
                        string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'AZURE_CLIENT_SECRET'), 
                        string(credentialsId: 'AZURE_TENANT_ID', variable: 'AZURE_TENANT_ID') 
                    ]) {
                        // Azure login using service principal
                        bat """
                            az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}
                        """
                    }
                }
            }
        }

        // Stage to deploy to Azure Web App
       stage('Deploy to Azure') {
            steps {
                script {
                    // Use Azure CLI to deploy to Azure Web App
                    bat """
                    az webapp deploy --resource-group SK --name jenkins-demo --src-path dist.zip
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up or other post steps
            echo 'Deployment process finished.'
        }
    }
}
