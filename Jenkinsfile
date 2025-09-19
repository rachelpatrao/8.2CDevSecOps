pipeline {
  agent any
  tools { nodejs 'node18' }   // NodeJS tool name must match what you configured in Jenkins

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
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh 'npm audit --audit-level=high | tee test.log'
        }
      }
      post {
        always {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] TEST stage - ${currentBuild.currentResult}",
            body: "See console: ${env.BUILD_URL}console",
            attachLog: true,
            attachmentsPattern: 'test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh 'npm audit --json | tee security-scan.json'
        }
      }
      post {
        always {
          emailext(
            to: env.NOTIFY_EMAIL,
            subject: "[${env.JOB_NAME} #${env.BUILD_NUMBER}] SECURITY SCAN stage - ${currentBuild.currentResult}",
            body: "See console: ${env.BUILD_URL}console",
            attachLog: true,
            attachmentsPattern: 'security-scan.json'
          )
        }
      }
    }

    stage('Static Analysis') {
      when { expression { fileExists('package.json') } }
      steps {
        sh 'npx eslint . || true'
      }
    }

    stage('Package') {
      steps {
        sh 'npm pack || true'
      }
    }

    stage('Deploy (demo)') {
      when { branch 'main' }
      steps {
        echo 'Pretend deploy…'
      }
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
