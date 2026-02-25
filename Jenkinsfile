pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running dummy tests..."
            }
        }

        stage('Publish Test Report') {
            steps {
                junit 'reports/junit.xml'
            }
        }

        stage('Notify GitHub') {
            steps {
                withCredentials([string(credentialsId: 'github-token-priyanshu', variable: 'GITHUB_TOKEN')]) {
                    script {

                        def status = "success"
                        def description = "All tests passed"

                        if (currentBuild.currentResult == "UNSTABLE") {
                            status = "failure"
                            description = "Some tests failed"
                        }

                        sh """
                        curl -X POST \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        https://api.github.com/repos/Priyanshu498/pinelabs/statuses/${GIT_COMMIT} \
                        -d '{
                          "state": "${status}",
                          "context": "ci/jenkins",
                          "description": "${description}"
                        }'
                        """
                    }
                }
            }
        }

    }
}
