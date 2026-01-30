pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    SONAR_PROJECT_KEY = "project"
    SONAR_HOST_URL    = "http://host.docker.internal:9000"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        sh '''
          node -v || true
          npm -v || true
          npm ci || npm install
        '''
      }
    }

    stage('Code Quality (SonarQube)') {
      steps {
        script {
          def scannerHome = tool 'SonarScanner'

          withSonarQubeEnv('sonar') {

            // IMPORTANT:
            // Jenkins → Manage Jenkins → Credentials
            // Add Secret Text
            // ID = sonar-token
            // Value = your SonarQube generated token

            withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {

              sh """
                ${scannerHome}/bin/sonar-scanner \
                  -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=${SONAR_HOST_URL} \
                  -Dsonar.login=$SONAR_TOKEN \
                  -Dsonar.javascript.node.maxspace=4096 \
                  -Dsonar.exclusions=node_modules/**,coverage/**,dist/**,build/**,**/*.min.js,**/public/js/ga.js
              """
            }
          }
        }
      }
      post {
        failure {
          echo "SonarQube failed but pipeline will continue..."
        }
      }
    }

    stage('Run Tests') {
      steps {
        sh '''
          if npm run 2>/dev/null | grep -q " test"; then
            echo "Running npm test (non-blocking)..."
            npm test || true
          else
            echo "No test script found. Skipping."
          fi
        '''
      }
    }

    stage('Snyk Security Scan') {
      steps {
        script {
          try {

            // OPTIONAL:
            // Jenkins → Credentials
            // Add Secret Text
            // ID = snyk-token
            // Value = your Snyk API token

            withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {

              sh '''
                echo "Running Snyk scan (non-blocking)..."
                npx snyk --version || true
                npx snyk test || true
              '''
            }

          } catch (err) {
            echo "Snyk credentials not found. Skipping Snyk stage."
          }
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        sh '''
          if npm run 2>/dev/null | grep -q " coverage"; then
            echo "Running coverage..."
            npm run coverage || true
          else
            echo "Coverage script not found. Skipping."
          fi
        '''
      }
    }

    stage('NPM Audit') {
      steps {
        sh '''
          echo "Running npm audit (non-blocking)..."
          npm audit || true
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Check Jenkins console + SonarQube dashboard."
    }

    success {
      emailext(
        subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build SUCCESS ✅
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
""",
        to: "rohithumi@gmail.com"
      )
    }

    failure {
      emailext(
        subject: "Jenkins Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build FAILED ❌
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
""",
        to: "rohithumi@gmail.com"
      )
    }
  }
}
