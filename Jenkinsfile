import groovy.json.JsonSlurper

def getAcrLoginServer(def acrSettingsJson) {
  def acrSettings = new JsonSlurper().parseText(acrSettingsJson)
  return acrSettings.loginServer
}

pipeline{
    agent any
    stages{
      stage('init') {
        steps {
          checkout scm
        }  
      }
      stage('Build image') {
        agent { dockerfile { dir 'calculator-api' } }
        steps {
          echo 'Building in progress'
        }
      }
    }
}

