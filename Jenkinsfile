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

    }

    post {
        always {
            script {

                def status = "SUCCESS"
                def description = "All tests passed"

                if (currentBuild.currentResult == "UNSTABLE") {
                    status = "FAILURE"
                    description = "Some tests failed"
                }

                githubNotify(
                    credentialsId: 'github-token-priyanshu',   // EXACT same ID
                    account: 'Priyanshu498',
                    repo: 'pinelabs',
                    sha: env.GIT_COMMIT,
                    context: 'ci/jenkins',
                    status: status,
                    description: description
                )
            }
        }
    }
}
