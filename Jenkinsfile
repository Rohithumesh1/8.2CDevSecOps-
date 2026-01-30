pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    SONAR_PROJECT_KEY = "project"
    SONAR_HOST_URL = "http://host.docker.internal:9000"
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
          npm install
        '''
      }
    }

    stage('Code Quality (SonarQube)') {
      steps {
        script {
          def scannerHome = tool 'SonarScanner'
          withSonarQubeEnv('sonar') {
            sh """
              ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                -Dsonar.sources=. \
                -Dsonar.host.url=${SONAR_HOST_URL} \
                -Dsonar.login=$SONAR_AUTH_TOKEN \
                -Dsonar.javascript.node.maxspace=4096 \
                -Dsonar.exclusions=node_modules/**,coverage/**,dist/**,build/**,**/*.min.js,**/public/js/ga.js
            """
          }
        }
      }
      post {
        unsuccessful {
          echo "SonarQube failed, but continuing pipeline..."
        }
      }
    }

    stage('Run Tests') {
      steps {
        sh '''
          if npm run | grep -q " test"; then
            echo "Running npm test..."
            npm test || true
          else
            echo "No test script found. Skipping."
          fi
        '''
      }
    }

    stage('Snyk Test (Security Scan)') {
      steps {
        script {
          withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
            sh '''
              echo "Running Snyk test..."
              npx snyk test || true
            '''
          }
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        sh '''
          if npm run | grep -q " coverage"; then
            npm run coverage || true
          else
            echo "Coverage not configured. Skipping."
          fi
        '''
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh '''
          echo "Running npm audit..."
          npm audit || true
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Check Sonar + Jenkins dashboard."
    }

    success {
      emailext(
        subject: "Jenkins SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build SUCCESS

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
""",
        to: "rohithumi@gmail.com"
      )
    }

    failure {
      emailext(
        subject: "Jenkins FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build FAILED

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
""",
        to: "rohithumi@gmail.com"
      )
    }
  }
}
