/** Desired capabilities */
def capabilities = [
  browserName: 'Chrome',
  platform: 'Windows 10'
]

pipeline {
  agent any
  libraries {
    lib('fxtest@1.9')
  }
  options {
    ansiColor('xterm')
    timestamps()
    timeout(time: 30, unit: 'MINUTES')
  }
  environment {
    PYTEST_PROCESSES = "${PYTEST_PROCESSES ?: "auto"}"
    PYTEST_ADDOPTS =
      "-n=${PYTEST_PROCESSES} " +
      "--tb=short " +
      "--color=yes " +
      "--driver=SauceLabs " +
      "--variables=capabilities.json"
    PULSE = credentials('PULSE')
    SAUCELABS_API_KEY = credentials('SAUCELABS_API_KEY')
  }
  stages {
    stage('Lint') {
      steps {
        sh "tox -e flake8"
      }
    }
    stage('Test') {
      steps {
        writeCapabilities(capabilities, 'capabilities.json')
        sh "tox -e py27"
      }
      post {
        always {
          archiveArtifacts 'results/*'
          junit 'results/*.xml'
          submitToActiveData('results/py27_raw.txt')
          submitToTreeherder('stubattribution-tests', 'e2e', 'End-to-end integration tests', 'results/*', 'results/py27_tbpl.txt')
          publishHTML(target: [
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'results',
            reportFiles: "py27.html",
            reportName: 'HTML Report'])
        }
      }
    }
  }
  post {
    failure {
      emailext(
        attachLog: true,
        attachmentsPattern: 'results/py27.html',
        body: '$BUILD_URL\n\n$FAILED_TESTS',
        replyTo: '$DEFAULT_REPLYTO',
        subject: '$DEFAULT_SUBJECT',
        to: '$DEFAULT_RECIPIENTS')
    }
    changed {
      ircNotification()
    }
  }
}
