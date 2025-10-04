pipeline {
  agent any

  environment {
    TOMCAT_HOST = 'localhost'    // change to remote host if Tomcat is remote
    TOMCAT_PORT = '8080'
    TOMCAT_MANAGER_PATH = '/manager/text'
    APP_PATH = '/welcomeapp'     // context path: http://TOMCAT_HOST:8080/welcomeapp
    CREDENTIALS_ID = 'tomcat-creds' // set this credential ID in Jenkins (username/password)
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'target/*.war', fingerprint: true
      }
    }

    stage('Deploy') {
      steps {
        script {
          // find the WAR file
          def war = sh(returnStdout: true, script: "ls -1 target/*.war | head -n1").trim()
          echo "WAR to deploy: ${war}"

          // Option A: Deploy via Tomcat Manager (recommended when Tomcat has manager-script user)
          withCredentials([usernamePassword(credentialsId: env.CREDENTIALS_ID, usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
            sh """
              echo "Uploading ${war} to Tomcat manager on ${TOMCAT_HOST}:${TOMCAT_PORT}"
              curl --fail --upload-file "${war}" "http://${TOMCAT_USER}:${TOMCAT_PASS}@${TOMCAT_HOST}:${TOMCAT_PORT}${TOMCAT_MANAGER_PATH}/deploy?path=${APP_PATH}&update=true"
            """
          }

          // Option B (alternate): If Jenkins runs on same machine and has permissions, copy to webapps:
          // sh "cp ${war} /opt/tomcat/webapps/welcomeapp.war"
        }
      }
    }
  }

  post {
    success {
      echo "Deployment stage finished. Visit: http://${env.TOMCAT_HOST}:${env.TOMCAT_PORT}${env.APP_PATH}"
    }
    failure {
      echo "Pipeline failed. Check console output."
    }
  }
}
