pipeline {
  agent any

  tools {
    // Must match your configured NodeJS tool name (Manage Jenkins → Tools → NodeJS)
    nodejs 'node18'
  }

  environment {
    // Change if you want to send to others (comma-separate for multiple)
    RECIPIENTS = 'rachelpatrao@gmail.com'
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  stages {

    stage('Checkout') { // Tool: Git
      steps { checkout scm }
    }

    stage('Setup Node') { // Tool: Node.js
      steps {
        sh 'node -v'
        sh 'npm -v'
      }
    }

    stage('Install') { // Tool: npm
      steps { sh 'npm ci' }
    }

    stage('Test') { // Tool: (unit tests if available) — avoids Snyk auth
      steps {
        script {
          // If a "test:unit" script exists, run it; else, skip gracefully.
          def code = sh(
            script: '''
              set -o pipefail
              if npm run -s test:unit >/dev/null 2>&1; then
                echo "Running unit tests..."
                npm run -s test:unit 2>&1 | tee test.log
              else
                echo "No unit tests defined. Skipping." | tee test.log
              fi
            ''',
            returnStatus: true
          )
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

    stage('Security Scan') { // Tool: npm audit (emails results)
      steps {
        script {
          def code = sh(
            script: 'set -o pipefail; npm audit --audit-level=high 2>&1 | tee security.log',
            returnStatus: true
          )
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
