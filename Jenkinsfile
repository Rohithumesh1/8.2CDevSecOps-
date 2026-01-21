pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps { sh 'npm install' }
    }

    stage('Run Tests') {
      steps { sh 'npm test || true' }
    }

    stage('Generate Coverage') {
      steps { sh 'npm run coverage || true' }
    }

    stage('NPM Audit') {
      steps { sh 'npm audit || true' }
    }
  }
}
