pipeline {
    agent any

    environment {
        SONARQUBE_INSTANCE = 'SonarQube'
        PROJECT_KEY = 'code-review'
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/shivasaiteja123/my-first-repo.git'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('code review') // You created this Jenkins credential
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_INSTANCE}") {
                    bat "\"C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner.bat\" " +
                        "-Dsonar.projectKey=${PROJECT_KEY} " +
                        "-Dsonar.sources=. " +
                        "-Dsonar.token=${SONAR_TOKEN}"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Fetch SonarQube Results') {
            steps {
                echo "Fetching analysis results..."
                // Placeholder if you want to save results later
            }
        }

        stage('Email Notification') {
            environment {
                MAILGUN_API_KEY = credentials('MailgunAPI')
                MAILGUN_DOMAIN = credentials('Mailgundomain')
            }
            steps {
                script {
                    def subject = "SonarQube Quality Gate Passed ✅"
                    def body = """
                    Hello,

                    Your code analysis for the project '${PROJECT_KEY}' passed the quality gate successfully.

                    View the report: ${SONAR_HOST_URL}/dashboard?id=${PROJECT_KEY}

                    Regards,
                    Jenkins CI/CD
                    """
                    def cmd = """
                    curl -s --user 'api:${MAILGUN_API_KEY}' \\
                        https://api.mailgun.net/v3/${MAILGUN_DOMAIN}/messages \\
                        -F from='Jenkins CI <mailgun@${MAILGUN_DOMAIN}>' \\
                        -F to='you@example.com' \\
                        -F subject='${subject}' \\
                        -F text='${body}'
                    """
                    bat label: 'Send Email via Mailgun', script: cmd
                }
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: '**/.scannerwork/report*', onlyIfSuccessful: true
            }
        }

        stage('Cleanup SonarQube Project') {
            environment {
                SONAR_TOKEN = credentials('SONAR_AUTH_TOKEN')
            }
            steps {
                script {
                    def curlCmd = """
                    curl -X POST -u ${SONAR_TOKEN}: \\
                    "${SONAR_HOST_URL}/api/projects/delete?project=${PROJECT_KEY}"
                    """
                    bat label: 'Delete SonarQube Project', script: curlCmd
                }
            }
        }
    }
}
