pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Rohithumesh1/8.2CDevSecOps-.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Code Quality (SonarQube)') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=project \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true'
            }
            post {
                always {
                    emailext(
                        subject: "Jenkins: TEST Stage Completed - ${currentBuild.currentResult} (${env.JOB_NAME} #${env.BUILD_NUMBER})",
                        body: """<p>Test stage finished.</p>
                                 <p><b>Status:</b> ${currentBuild.currentResult}</p>
                                 <p><b>Job:</b> ${env.JOB_NAME} #${env.BUILD_NUMBER}</p>
                                 <p><b>Build URL:</b> ${env.BUILD_URL}</p>""",
                        to: "rohithumi@gmail.com",
                        attachLog: true
                    )
                }
            }
        }

        stage('Generate Coverage Report') {
            steps {
                sh 'npm run coverage || true'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                sh 'npm audit || true'
            }
            post {
                always {
                    emailext(
                        subject: "Jenkins: SECURITY SCAN Stage Completed - ${currentBuild.currentResult} (${env.JOB_NAME} #${env.BUILD_NUMBER})",
                        body: """<p>Security scan stage finished.</p>
                                 <p><b>Status:</b> ${currentBuild.currentResult}</p>
                                 <p><b>Job:</b> ${env.JOB_NAME} #${env.BUILD_NUMBER}</p>
                                 <p><b>Build URL:</b> ${env.BUILD_URL}</p>
                                 <p>Check console log attachment for <code>npm audit</code> findings.</p>""",
                        to: "rohithumi@gmail.com",
                        attachLog: true
                    )
                }
            }
        }
    }
}
