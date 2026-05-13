pipeline {
    agent any

    environment {
        VENV = 'venv'
    }

    stages {
        stage('Checkout') {
            steps {
                // Uses the SCM configuration defined in the Jenkins job
                checkout scm
            }
        }

        stage('Setup') {
            steps {
                script {
                    echo 'Initializing environment and installing dependencies...'
                    sh """
                        python3 -m venv ${VENV}
                        . ${VENV}/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        # Ensure black is installed for quality checks
                        pip install black
                    """
                }
            }
        }

        stage('Quality') {
            steps {
                script {
                    echo 'Checking code style with Black...'
                    sh ". ${VENV}/bin/activate && black --check ."
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Packaging project via setup.py...'
                    sh ". ${VENV}/bin/activate && python3 setup.py sdist bdist_wheel"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running tests with unittest...'
                    // Discovers and runs tests in the current directory
                    sh ". ${VENV}/bin/activate && python3 -m unittest discover"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deployment phase...'
                    // Placeholder for your deployment logic
                    archiveArtifacts artifacts: 'dist/*', fingerprint: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the unit test or quality logs.'
        }
        always {
            // Cleanup the environment
            sh "rm -rf ${VENV}"
        }
    }
}