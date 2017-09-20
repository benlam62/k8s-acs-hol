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
    def calculatorImageName = 'calculator'
    def calculatorDeploymentName = 'deployments/azure-calculator-api'
    def calculatorPodName = 'azure-calculator-api'
    def voteFrontImageName = 'azure-vote-front'
    def voteFrontDeploymentName = 'deployments/azure-vote-front'
    def voteFrontPodName = 'azure-vote-front'
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
      // build images
      agent { dockerfile { dir 'calculator-api' } }
      steps {
        def calculatorImageWithTag = "$loginServer/$calculatorImageName:$version"
        def calculatorImage = docker.build calculatorImageWithTag

      }
      agent { dockerfile { dir 'azure-vote' } }
      steps {
        def voteFrontImageWithTag = "$loginServer/$voteFrontImageName:$version"
        def voteFrontImage = docker.build voteFrontImageWithTag
      }
      // push images
      calculaterImage.push()
      voteFrontImage.push()
      // update web app docker settings
      //sh "az webapp config container set -g $webAppResourceGroup -n $webAppName -c $imageWithTag -r http://$loginServer -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET"
      // update K8s cluster
      sh "kubectl set image $calculatorDeploymentName $calculatorPodName=$calculatorImageWithTag"
      sh "kubectl set image $voteFrontDeploymentName $voteFrontPodName=$voteFrontImageWithTag"
      // log out
      sh 'az logout'
      sh "docker logout $loginServer"
    }
  }
}
