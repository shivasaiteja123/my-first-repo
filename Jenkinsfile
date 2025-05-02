pipeline {
    agent any

    environment {
        SONARQUBE_ENV = credentials('code review')
        SONAR_AUTH_TOKEN = credentials('sonar-token')
        MAILGUN_API_KEY = credentials('mailgun-api-key')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/shivasaiteja123/my-first-repo.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        bat "\"C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner.bat\" -Dsonar.projectKey=code-review -Dsonar.sources=. -Dsonar.token=${SONAR_AUTH_TOKEN}"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Fetch SonarQube Results') {
            steps {
                echo 'Fetching analysis results...'
                script {
                    def response = httpRequest(
                        url: 'http://localhost:9000/api/measures/component?component=code-review&metricKeys=bugs,vulnerabilities,code_smells,coverage',
                        httpMode: 'GET',
                        customHeaders: [[name: 'Authorization', value: "Basic ${SONAR_AUTH_TOKEN.bytes.encodeBase64().toString()}"]],
                        validResponseCodes: '200'
                    )
                    writeFile file: 'sonar-results.json', text: response.content
                }
            }
        }

        stage('Send Email Notification') {
            steps {
                script {
                    def results = readJSON file: 'sonar-results.json'
                    def measures = results.component.measures.collect { "${it.metric}: ${it.value}" }.join('<br>')
                    def subject = "SonarQube Analysis Report for code-review"
                    def body = """
                    <h3>SonarQube Analysis Results:</h3>
                    <p>${measures}</p>
                    <p>Dashboard: <a href="http://localhost:9000/dashboard?id=code-review">View Here</a></p>
                    """

                    sh """
                    curl -s --user "api:${MAILGUN_API_KEY}" \\
                        https://api.mailgun.net/v3/YOUR_DOMAIN_NAME/messages \\
                        -F from="CI Pipeline <mailgun@YOUR_DOMAIN_NAME>" \\
                        -F to="you@example.com" \\
                        -F subject="${subject}" \\
                        -F html="${body}"
                    """
                }
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'sonar-results.json', onlyIfSuccessful: true
            }
        }

        stage('Clean Up SonarQube') {
            steps {
                echo 'Cleaning up project from SonarQube...'
                script {
                    httpRequest(
                        url: 'http://localhost:9000/api/projects/delete?project=code-review',
                        httpMode: 'POST',
                        customHeaders: [[name: 'Authorization', value: "Basic ${SONAR_AUTH_TOKEN.bytes.encodeBase64().toString()}"]],
                        validResponseCodes: '200'
                    )
                }
            }
        }
    }
}
