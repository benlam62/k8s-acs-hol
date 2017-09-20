pipeline{
    agent any
    stages{
      stage('init') {
        step {
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

