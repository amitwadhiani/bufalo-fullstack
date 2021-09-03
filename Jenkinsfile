
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
    stage('Db Creation') {     
      withCredentials([string(credentialsId: 'sshHostPassSecret', variable: 'pass'), string(credentialsId: 'databasePassword', variable: 'dbPass')] {
        sh"""
        sshpass -p $pass ssh root@188.166.87.169
        echo "Login to Host successfull"
        docker run --name rw_db -e POSTGRES_DB=gobuff_realworld_example_app_development -e POSTGRES_PASSWORD=$dbPass -e POSTGRES_USER=postgres -p 5432:5432 -d postgres
        sleep 30
        """
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
              az deployment group create --resource-group \$resourceGroup  --name \$deploymentName --template-file azureDeploy.json --parameters dev.parameters.json --parameters dockerRegistryPassword=${dockerPassword} --parameters linuxFxVersion="DOCKER|amit0wadhiani/bufalo-fullstack:${env.BUILD_NUMBER}" --parameters databasePassword=${databasePassword} --parameters dockerRegistryPassword=${dockerPassword} --parameters databaseUrl='postgres://postgres:${databasePassword}@188.166.87.169:5432/gobuff_realworld_example_app_production?sslmode=disable' --parameters testDatabaseUrl='postgres://postgres:${databasePassword}@188.166.87.169:5432/gobuff_realworld_example_app_production?sslmode=disable' --parameters databaseHost='188.166.87.169'
            """
          }
      }
  }