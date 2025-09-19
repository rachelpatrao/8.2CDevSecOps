pipeline {
  agent any
  tools { nodejs 'node18' }   // Remove if not using the NodeJS plugin

  environment {
    NOTIFY_EMAIL = 'rachelpatrao@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        // Run tests and capture logs (pipeline continues even if tests fail)
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh 'npm test | tee test.log'
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'test.log', allowEmptyArchive: true
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] TEST stage - ${currentBuild.currentResult}",
            body: """<p>Test stage finished with status: ${currentBuild.currentResult}</p>
                     <p><a href="${env.BUILD_URL}console">Open console log</a></p>
                     <p>See attached test.log for details.</p>""",
            attachmentsPattern: 'test.log',
            mimeType: 'text/html'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        // Run security scan (Snyk if token available, else npm audit)
        script {
          if (env.SNYK_TOKEN) {
            sh '''
              npx snyk auth $SNYK_TOKEN
              npx snyk test --severity-threshold=high | tee security.log || true
            '''
          } else {
            sh 'npm audit --audit-level=high | tee security.log || true'
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'security.log', allowEmptyArchive: true
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] SECURITY SCAN - ${currentBuild.currentResult}",
            body: """<p>Security scan finished with status: ${currentBuild.currentResult}</p>
                     <p><a href="${env.BUILD_URL}console">Open console log</a></p>
                     <p>See attached security.log for details.</p>""",
            attachmentsPattern: 'security.log',
            mimeType: 'text/html'
          )
        }
      }
    }
  }
}
