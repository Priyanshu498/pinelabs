pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Publish Test Report') {
            steps {
                junit 'reports/junit.xml'

                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'report.html',
                    reportName: 'CI Test Report'
                ])
            }
        }

        stage('Notify GitHub') {
            steps {
                withCredentials([string(credentialsId: 'github-token-priyanshu', variable: 'GITHUB_TOKEN')]) {
                    script {

                        def status = (currentBuild.currentResult == "UNSTABLE") ? "failure" : "success"

                        def reportUrl = "${env.BUILD_URL}CI_20Test_20Report/"

                        sh """
                        curl -X POST \
                        -H "Authorization: token $GITHUB_TOKEN" \
                        -H "Accept: application/vnd.github.v3+json" \
                        https://api.github.com/repos/Priyanshu498/pinelabs/statuses/$GIT_COMMIT \
                        -d '{
                          "state": "${status}",
                          "context": "ci/jenkins",
                          "description": "View Test Report",
                          "target_url": "${reportUrl}"
                        }'
                        """
                    }
                }
            }
        }

    }
}
