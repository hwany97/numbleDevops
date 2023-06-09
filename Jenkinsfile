pipeline {
    agent any
    environment {
        AZURE_STORAGE_ACCOUNT='ikstorageexample'
    }
    stages {
        stage("Checkout Repo") {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/diogenes0803/ik-cicd.git']]])
            }
        }
        stage("Build") {
            steps {
                nodejs('node18') {
                    sh 'npm install'
                    sh 'npm run build'
                    sh 'tar -zcf ${JOB_NAME}${BUILD_NUMBER}.tar.gz ./build/*'
                }
            }
            post {
                success {
                    sh '''
                      # Login to Azure with ServicePrincipal
                      az login --identity
                      # Execute upload to Azure
                      az storage container create --account-name $AZURE_STORAGE_ACCOUNT --name $JOB_NAME --auth-mode login
                      az storage blob upload --auth-mode login -c ${JOB_NAME} -f ./${JOB_NAME}${BUILD_NUMBER}.tar.gz -n ${JOB_NAME}${BUILD_NUMBER}.tar.gz --account-name $AZURE_STORAGE_ACCOUNT
                      # Logout from Azure
                      az logout
                    '''
                }
          }
        }
        stage("Deploy") {
            steps {
                sh '''
                    ansible-playbook -i ./ansible/deploy/web.ini ./ansible/deploy/deploy_web_server_playbook.yml --extra-vars "container_name='${JOB_NAME}' file_name='${JOB_NAME}${BUILD_NUMBER}.tar.gz'"
                '''
            }
        }
    }
}


