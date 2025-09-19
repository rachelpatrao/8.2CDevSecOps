pipeline {
  agent any

  environment {
    // Change this to the email address that should receive notifications
    NOTIFY_EMAIL = 'rachelpatrao@gmail.com'
  }

  options {
    timestamps()
    // Keep the console log; emails will attach it (gzipped)
    ansiColor('xterm')
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
      // (Tool ref: Node.js runtime)
    }

    stage('Install') {
      steps {
        sh 'npm ci'
      }
      // (Tool ref: npm for dependency install)
    }

    stage('Test') {
      steps {
        // Run your unit tests (adjust if your project uses mocha/jest etc.)
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
        always {
          // You can also add attachmentsPattern here if you export JUnit/XML, e.g.:
          // attachmentsPattern: '**/junit*.xml'
        }
      }
      // (Tool ref: Jest/Mocha for tests)
    }

    stage('Security Scan') {
      steps {
        // Prefer Snyk if you have it; fallback to npm audit
        sh '''
          if command -v snyk >/dev/null 2>&1; then
            echo "Running Snyk test..."
            snyk test || true
          else
            echo "Snyk not found; running npm audit (non-blocking)..."
            npm audit --audit-level=high || true
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
        always {
          // If you export Snyk JSON, you can attach it like:
          // attachmentsPattern: '**/snyk-report.json'
        }
      }
      // (Tool ref: Snyk CLI or npm audit)
    }

    // Optional examples (not required by your Task 2, but handy)

    stage('Static Analysis') {
      when { expression { fileExists('package.json') } }
      steps {
        sh 'npx eslint . || true'
      }
      // (Tool ref: ESLint)
    }

    stage('Package') {
      steps {
        sh 'npm pack || true'
      }
      // (Tool ref: npm pack, or Docker for container packaging)
    }

    stage('Deploy (demo)') {
      when { branch 'main' }
      steps {
        echo 'Pretend deploy…'
        // e.g., Docker/K8s/Heroku/GitHub Pages—adapt to your real target
      }
      // (Tool ref: Docker CLI/Kubernetes/Heroku CLI/GitHub Actions)
    }
  }

  // Optional: overall build notifications
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
