pipeline {
  agent any
  environment {
    NOTIFY_EMAIL = 'rachelpatrao@gmail.com'
  }
  stages {
    stage('Dummy') {
      steps {
        echo 'Just testing email...'
      }
    }
  }
  post {
    always {
      emailext(
        to: env.NOTIFY_EMAIL,
        subject: "Test Email from Jenkins",
        body: "If you got this, email setup works!",
        attachLog: true
      )
    }
  }
}
