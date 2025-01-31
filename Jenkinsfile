pipeline {
    agent any

    environment {
        NODE_HOME = tool 'nodejs'  // Use the NodeJS configured in Jenkins
        PATH = "${NODE_HOME}/bin:${env.PATH}"
        NEXUS_URL = 'http://localhost:8081/repository/dist/'  // Nexus Repository URL
        NEXUS_USER = 'admin'  // Nexus Username
        NEXUS_PASSWORD = 'vamsi@123'  // Nexus Password
        ARTIFACT_NAME = 'middlewaretalents' // Artifact Name
        ARTIFACT_VERSION = '1.0.1' // Artifact Version
        GROUP_ID = 'com.middlewaretalents' // Artifact Group ID
        NGINX_PATH = 'C:\\Users\\MTL1020\\Downloads\\nginx-1.26.2\\nginx-1.26.2\\html'  // Path to Nginx HTML directory
        AZURE_RESOURCE_GROUP = 'eshwar'  // Azure Resource Group (Change this)
        AZURE_APP_NAME = 'eshwar-test'  // Azure Web App Name (Change this)
        ZIP_FILE = "${ARTIFACT_NAME}-${ARTIFACT_VERSION}.zip"  // Zip file for Azure Web App deployment
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/vamsimohanyacham/test.git', credentialsId: '2f167f4e-84fd-4929-8d9a-ba8f849897bd'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Build Vite App') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Check Build Directory') {
            steps {
                bat 'dir dist'
            }
        }

        stage('Lint Code') {
            steps {
                bat 'npm run lint'
            }
        }

        stage('Check Code Formatting') {
            steps {
                bat 'npm run format'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm run test'
            }
        }

        stage('Create .zip Archive') {
            steps {
                script {
                    bat 'dir dist'
                    bat "powershell Compress-Archive -Path dist\\* -DestinationPath ${ARTIFACT_NAME}-${ARTIFACT_VERSION}.zip"
                    echo "Created ${ARTIFACT_NAME}-${ARTIFACT_VERSION}.zip from dist folder"
                    bat 'dir'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    def artifactFile = "${ARTIFACT_NAME}-${ARTIFACT_VERSION}.zip"
                    def nexusRepositoryUrl = "${NEXUS_URL}${ARTIFACT_NAME}-${ARTIFACT_VERSION}.zip"
                    bat """
                        echo Uploading ${artifactFile} to Nexus...
                        curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} -X PUT -F "file=@${artifactFile}" ${nexusRepositoryUrl}
                    """
                }
            }
        }

        stage('Download Artifact from Nexus') {
            steps {
                script {
                    def artifactFile = "${ARTIFACT_NAME}-${ARTIFACT_VERSION}.zip"
                    echo "Downloading ${artifactFile} from Nexus..."
                    bat """
                        curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} -O ${NEXUS_URL}${artifactFile}
                    """
                }
            }
        }

        stage('Extract Artifact') {
            steps {
                script {
                    bat 'dir'
                    bat "powershell Expand-Archive -Path ${ARTIFACT_NAME}-${ARTIFACT_VERSION}.zip -DestinationPath ."
                    echo "Artifact extracted"
                }
            }
        }

        stage('Move dist to Nginx Directory') {
            steps {
                script {
                    bat "xcopy /E /I /H /Y dist ${NGINX_PATH}\\"
                    echo "Moved dist folder to Nginx directory"
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

        stage('Deploy to Azure') {
            steps {
                script {
                    // Deploy the zip file to Azure Web App using updated command
                    bat """
                        az webapp deploy --resource-group ${AZURE_RESOURCE_GROUP} --name ${AZURE_APP_NAME} --src ${ZIP_FILE}
                    """
                }
            }
        }
    }

    post {
        always {
            bat 'del /F /Q *.zip || true'
        }
        
        success {
            // Send email on successful deployment
            emailext (
                subject: "Deployment Successful: ${ARTIFACT_NAME}-${ARTIFACT_VERSION}",
                body: "The deployment of the artifact ${ARTIFACT_NAME}-${ARTIFACT_VERSION} was successful! You can now check the Nginx server and Azure Web App to verify the update.",
                to: 'vamsimohanyacham@gmail.com',  // Ensure recipient is specified
                from: 'yaswanthkumarch2001@gmail.com'
            )
        }

        failure {
            // Send email on failure
            emailext (
                subject: "Deployment Failed: ${ARTIFACT_NAME}-${ARTIFACT_VERSION}",
                body: "The deployment of the artifact ${ARTIFACT_NAME}-${ARTIFACT_VERSION} has failed. Please check the Jenkins logs for details.",
                to: 'vamsimohanyacham@gmail.com',  // Ensure recipient is specified
                from: 'yaswanthkumarch2001@gmail.com'
            )
        }
    }
}
