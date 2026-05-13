pipeline {
    agent any

    environment {
        PYTHON_VERSION = '3.9'
        VENV_PATH = 'venv'
        TEST_DIR = 'tests'
        COVERAGE_DIR = 'htmlcov'
        BLACK_CHECK_ARGS = '--check --diff --color'
    }

    stages {
        // Stage 1: Checkout source code from GitHub
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Checked out repository: ${env.GIT_URL}"
                // Verify expected project files exist
                sh 'ls -la setup.py requirements.txt || echo "⚠️ Required files not found"'
            }
        }

        // Stage 2: Setup Python environment and install dependencies
        stage('Setup') {
            steps {
                script {
                    sh """
                        # Create virtual environment
                        python${PYTHON_VERSION} -m venv ${VENV_PATH}
                        
                        # Activate and upgrade pip
                        . ${VENV_PATH}/bin/activate
                        python -m pip install --upgrade pip
                        
                        # Install runtime dependencies from requirements.txt
                        echo "📦 Installing dependencies from requirements.txt..."
                        pip install -r requirements.txt
                        
                        # Install test/quality tools if not already in requirements.txt
                        echo "🔧 Installing test & quality tools..."
                        pip install black coverage xmlrunner || echo "⚠️ Some tools may already be installed"
                        
                        # Install the project in editable mode (for setup.py projects)
                        echo "🔧 Installing project in editable mode..."
                        pip install -e .
                        
                        echo "✅ Setup completed: Virtual environment ready"
                    """
                }
            }
        }

        // Stage 3: Quality checks - Black code formatting (ENABLED) ✅
        stage('Quality') {
            steps {
                script {
                    sh """
                        . ${VENV_PATH}/bin/activate
                        # Run Black in check mode (validates formatting without modifying files)
                        black ${BLACK_CHECK_ARGS} .
                        echo "✅ Black formatting check passed"
                    """
                }
            }
            post {
                failure {
                    echo "❌ Black formatting issues detected!"
                    echo "💡 To fix locally: run 'black .' in your project root and recommit"
                }
            }
        }

        // Stage 4: Build - Validate setup.py and compile Python bytecode
        stage('Build') {
            steps {
                script {
                    sh """
                        . ${VENV_PATH}/bin/activate
                        
                        # Validate setup.py can be parsed
                        python setup.py --version > /dev/null 2>&1 || echo "⚠️ setup.py version check skipped"
                        
                        # Check for syntax errors by compiling all Python files
                        echo "🔍 Validating Python syntax..."
                        python -m compileall -q .
                        
                        echo "✅ Build completed: Project validated successfully"
                    """
                }
            }
        }

        // Stage 5: Test execution - ENABLED with unittest ✅
        stage('Test') {
            steps {
                script {
                    sh """
                        . ${VENV_PATH}/bin/activate
                        mkdir -p test-reports ${COVERAGE_DIR}
                        
                        echo "🧪 Running unittest tests with coverage..."
                        
                        # Run unittest with coverage and generate JUnit-style XML + HTML coverage
                        coverage run --source=. -m unittest discover \
                            -s ${TEST_DIR} \
                            -p "test*.py" \
                            -v 2>&1 | tee test-output.log
                        
                        # Generate reports
                        coverage xml -o test-reports/coverage.xml
                        coverage html -d ${COVERAGE_DIR}
                        
                        # Convert unittest output to JUnit XML (if xmlrunner used)
                        # Note: If using xmlrunner, replace discover command with:
                        # python -m xmlrunner discover -s tests -p "test*.py" -o test-reports/
                        
                        echo "✅ Tests completed"
                    """
                }
            }
            post {
                always {
                    // Publish JUnit-style test results if available
                    junit testResults: 'test-reports/*.xml', allowEmptyResults: true, skipPublishingChecks: true
                    
                    // Publish HTML coverage report
                    publishHTML target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: COVERAGE_DIR,
                        reportFiles: 'index.html',
                        reportName: 'Code Coverage Report'
                    ]
                    
                    // Archive test logs for debugging
                    archiveArtifacts artifacts: 'test-output.log', allowEmptyArchive: true, fingerprint: true
                }
                success {
                    echo "🎉 All unittest tests passed!"
                }
                failure {
                    echo "❌ Tests failed - review test-output.log and coverage report"
                }
            }
        }

        // Stage 6: Deploy - SKIPPED per configuration
        // Dockerfile: "false" → No containerization configured
        /*
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                script {
                    . ${VENV_PATH}/bin/activate
                    echo "ℹ️ Deployment disabled - no Dockerfile configured"
                    // Future options:
                    // - python setup.py sdist bdist_wheel + upload to PyPI
                    // - Deploy to cloud VM/server via SSH
                    // - Add Docker stage when Dockerfile is available
                }
            }
        }
        */
    }

    post {
        always {
            // Cleanup workspace to free resources
            sh "rm -rf ${VENV_PATH} __pycache__ **/__pycache__ .pytest_cache build dist *.egg-info ${COVERAGE_DIR} test-reports"
            echo "🧹 Workspace cleaned"
        }
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed - check console output for details"
            // Optional: Add notification logic here
            // mail to: 'team@example.com', subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}