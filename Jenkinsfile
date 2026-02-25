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

                // Determine build status
                def status = "SUCCESS"
                def description = "All tests passed"

                if (currentBuild.result == "UNSTABLE") {
                    status = "FAILURE"
                    description = "Some tests failed"
                }

                // Notify GitHub
                githubNotify(
                    credentialsId: 'github-token-priyanshu',
                    repo: 'pinelabs',
                    account: 'Priyanshu498',
                    context: 'ci/jenkins',
                    status: status,
                    description: description,
                    sha: env.GIT_COMMIT
                )
            }
        }
    }
}
