pipeline {
  agent any

  tools {
    // Must match the name you configured under Manage Jenkins → Tools → NodeJS
    nodejs 'node18'
  }

  environment {
    // Change to your recipient(s); comma-separate for multiple
    RECIPIENTS = 'rachelpatrao@gmail.com'
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Setup Node') {
      steps {
        sh 'node -v'
        sh 'npm -v'
      }
    }

    stage('Install') {
      steps { sh 'npm ci' }
    }

    stage('Test') {
      steps {
        script {
          // Run tests, capture output to test.log, don’t fail the whole pipeline
          def code = sh(script: 'set -o pipefail; npm test 2>&1 | tee test.log', returnStatus: true)
          env.TEST_STATUS = (code == 0) ? 'SUCCESS' : 'FAILURE'
          if (code != 0) unstable('Tests failed.')
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'test.log', allowEmptyArchive: true
          emailext(
            to: env.RECIPIENTS,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] TEST stage - ${env.TEST_STATUS}",
            mimeType: 'text/html',
            body: """
              <p><b>Job:</b> ${env.JOB_NAME} #${env.BUILD_NUMBER}</p>
              <p><b>Stage:</b> ${env.STAGE_NAME}</p>
              <p><b>Status:</b> <b>${env.TEST_STATUS}</b></p>
              <p><b>Console:</b> <a href="${env.BUILD_URL}console">${env.BUILD_URL}console</a></p>
              <p>Attached: <code>test.log</code></p>
            """,
            attachmentsPattern: 'test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        script {
          // Default simple scan: npm audit; save to security.log
          def code = sh(script: 'set -o pipefail; npm audit --audit-level=high 2>&1 | tee security.log', returnStatus: true)
          env.SEC_STATUS = (code == 0) ? 'SUCCESS' : 'FAILURE'
          if (code != 0) unstable('Security scan found issues.')
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'security.log', allowEmptyArchive: true
          emailext(
            to: env.RECIPIENTS,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] SECURITY SCAN - ${env.SEC_STATUS}",
            mimeType: 'text/html',
            body: """
              <p><b>Job:</b> ${env.JOB_NAME} #${env.BUILD_NUMBER}</p>
              <p><b>Stage:</b> ${env.STAGE_NAME}</p>
              <p><b>Status:</b> <b>${env.SEC_STATUS}</b></p>
              <p><b>Console:</b> <a href="${env.BUILD_URL}console">${env.BUILD_URL}console</a></p>
              <p>Attached: <code>security.log</code></p>
            """,
            attachmentsPattern: 'security.log'
          )
        }
      }
    }
  }
}
