import groovy.json.JsonSlurper

def getAcrLoginServer(def acrSettingsJson) {
  def acrSettings = new JsonSlurper().parseText(acrSettingsJson)
  return acrSettings.loginServer
}

node {
  stage('init') {
    checkout scm
  }
  
  stage('build') {
    sh 'cd ./calculator-api && mvn clean package'
  }
  
  stage('deploy') {
    def webAppResourceGroup = 'linux-webapp'
    def webAppName = 'benlam-linux1'
    def acrName = 'benlamRegistry1'
    def imageName = 'calculator'
    def k8sDeploymentName = 'deployments/azure-calculator-api'
    def k8sPodName = 'azure-calculator-api'
    // generate version, it's important to remove the trailing new line in git describe output
    def version = sh script: 'git describe | tr -d "\n"', returnStdout: true
    withCredentials([azureServicePrincipal('e522eb25-3612-45b8-8248-586fe2e3eefe')]) {
      // login Azure
      sh '''
        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
        az account set -s $AZURE_SUBSCRIPTION_ID
      '''
      // get login server
      def acrSettingsJson = sh script: "az acr show -n $acrName", returnStdout: true
      def loginServer = getAcrLoginServer acrSettingsJson
      // login docker
      // docker.withRegistry only supports credential ID, so use native docker command to login
      // you can also use docker.withRegistry if you add a credential
      sh "whoami"
      sh "docker login -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET $loginServer"
      // build image
      def imageWithTag = "$loginServer/$imageName:$version"
      def image = docker.build ("$imageWithTag", "-f calculator-api/Dockerfile .")
      // push image
      image.push()
      // update web app docker settings
      //sh "az webapp config container set -g $webAppResourceGroup -n $webAppName -c $imageWithTag -r http://$loginServer -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET"
      // update K8s cluster
      sh "kubectl set image $k8sDeploymentName $k8sPodName=$imageWithTag"
      // log out
      sh 'az logout'
      sh "docker logout $loginServer"
    }
  }
}