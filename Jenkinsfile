import groovy.json.JsonSlurper

// 从发布配置 JSON 中提取 FTP 信息
def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
}

pipeline {
  agent any
  tools {
    maven 'Maven3'
  }
  environment {
    AZURE_SUBSCRIPTION_ID = 'f72101ce-beac-4ee0-ab42-ae97b7beec9c'
    AZURE_TENANT_ID       = '01748748-d987-4919-8e6a-7ea9f78f71a3'
  }
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
    stage('deploy') {
      steps {
        script {
          def resourceGroup = 'jenkins-get-started-rg'
          def webAppName   = 'zhenjenkinswebapp'

          withCredentials([usernamePassword(
            credentialsId: 'AzureServicePrincipal',
            passwordVariable: 'AZURE_CLIENT_SECRET',
            usernameVariable: 'AZURE_CLIENT_ID'
          )]) {
            sh '''
              az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
              az account set -s $AZURE_SUBSCRIPTION_ID
            '''
          }

          def pubProfilesJson = sh(
            script: "az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName}",
            returnStdout: true
          )
          def ftpProfile = getFtpPublishProfile(pubProfilesJson)

          sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"

          sh 'az logout'
        }
      }
    }
  }
}
