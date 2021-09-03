
node {
   def commit_id
   stage('Preparation') {
     checkout scm
     sh "git rev-parse --short HEAD > .git/commit-id"                        
     commit_id = readFile('.git/commit-id').trim()
    }  
   stage('Docker build/push') {     
     docker.withRegistry('https://index.docker.io/v1/', '998ffb10-3286-4316-9fad-ef16a26aaa52') {
       def app = docker.build("amit0wadhiani/bufalo-fullstack:${env.BUILD_NUMBER}", '.').push()
      }
     docker.withRegistry('https://index.docker.io/v1/', '998ffb10-3286-4316-9fad-ef16a26aaa52') {
       def app = docker.build("amit0wadhiani/bufalo-fullstack:${commit_id}", '.').push()
      }
    }
   stage('Deploy to App Service') {
        withCredentials([azureServicePrincipal('service-principal'), string(credentialsId: 'dockerPassword', variable: 'dockerPassword')]) {
          echo "Deploying to azure app service"
          sh """
              cd armTemplates
              resourceGroup='amitRG-dev'
              deploymentName='bufalo-fullstack'
              az login --service-principal --username ${AZURE_CLIENT_ID} --password ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID} # --subscription ${AZURE_SUBSCRIPTION_ID}
              az group create -l eastus2 -n \$resourceGroup --subscription ${AZURE_SUBSCRIPTION_ID}
              az deployment group create --resource-group \$resourceGroup  --name \$deploymentName --template-file azureDeploy.json --parameters dev.parameters.json --parameters dockerRegistryPassword=${dockerPassword} --parameters linuxFxVersion="DOCKER|amit0wadhiani/bufalo-fullstack:${env.BUILD_NUMBER}"
            """
          }
      }
  }