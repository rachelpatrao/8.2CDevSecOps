pipeline {
  agent any
  tools { nodejs 'Node18' }   // <-- adds Node & npm to PATH

  environment {
    NOTIFY_EMAIL = 'rachelpatrao@gmail.com'
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup Node') {
      steps {
        sh 'node -v'
        sh 'npm -v'
      }
    }

    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
      post {
        success {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] TEST stage: SUCCESS",
            mimeType: 'text/html',
            body: """
              <h3>Test Stage Completed</h3>
              <p><b>Status:</b> SUCCESS</p>
              <p><b>Job:</b> ${env.JOB_NAME}</p>
              <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
              <p><a href="${env.BUILD_URL}console">Open console log</a></p>
              <p>The full build log is attached.</p>
            """,
            attachLog: true,
            compressLog: true
          )
        }
        failure {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] TEST stage: FAILURE",
            mimeType: 'text/html',
            body: """
              <h3>Test Stage Completed</h3>
              <p><b>Status:</b> FAILURE</p>
              <p><b>Job:</b> ${env.JOB_NAME}</p>
              <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
              <p><a href="${env.BUILD_URL}console">Open console log</a></p>
              <p>The full build log is attached.</p>
            """,
            attachLog: true,
            compressLog: true
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        sh '''
          if command -v snyk >/dev/null 2>&1; then
            echo "Running Snyk test..."
            snyk test
          else
            echo "Snyk not found; running npm audit…"
            npm audit --audit-level=high
          fi
        '''
      }
      post {
        success {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] SECURITY SCAN stage: SUCCESS",
            mimeType: 'text/html',
            body: """
              <h3>Security Scan Stage Completed</h3>
              <p><b>Status:</b> SUCCESS</p>
              <p><b>Job:</b> ${env.JOB_NAME}</p>
              <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
              <p><a href="${env.BUILD_URL}console">Open console log</a></p>
              <p>The full build log is attached.</p>
            """,
            attachLog: true,
            compressLog: true
          )
        }
        failure {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] SECURITY SCAN stage: FAILURE",
            mimeType: 'text/html',
            body: """
              <h3>Security Scan Stage Completed</h3>
              <p><b>Status:</b> FAILURE</p>
              <p><b>Job:</b> ${env.JOB_NAME}</p>
              <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
              <p><a href="${env.BUILD_URL}console">Open console log</a></p>
              <p>The full build log is attached.</p>
            """,
            attachLog: true,
            compressLog: true
          )
        }
      }
    }

    // Optional stages...
    stage('Static Analysis') {
      when { expression { fileExists('package.json') } }
      steps {
        sh 'npx eslint . || true'
      }
    }
    stage('Package') {
      steps { sh 'npm pack || true' }
    }
    stage('Deploy (demo)') {
      when { branch 'main' }
      steps { echo 'Pretend deploy…' }
    }
  }

  post {
    failure {
      emailext(
        to: env.NOTIFY_EMAIL,
        subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] BUILD: FAILURE",
        body: "Overall build failed. See console: ${env.BUILD_URL}console",
        attachLog: true,
        compressLog: true
      )
    }
  }
}
