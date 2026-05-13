pipeline {
    agent any
    
    tools {
        python 'Python3'
    }
    
    environment {
        VENV_DIR = 'venv'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Checking out code...'
                checkout scm
            }
        }
        
        stage('Setup') {
            steps {
                echo '🐍 Setting up Python environment...'
                sh '''
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    
                    pip install --upgrade pip setuptools wheel
                    
                    # Install dependencies from requirements.txt
                    if [ -f requirements.txt ]; then
                        pip install -r requirements.txt
                    fi
                    
                    # Install the project in editable mode
                    pip install -e .
                    
                    echo "✅ Setup completed successfully"
                '''
            }
        }
        
        stage('Build') {
            steps {
                echo '🏗️ Building Python package...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    python setup.py build
                    echo "✅ Build completed"
                '''
            }
        }
        
        stage('Quality') {
            steps {
                echo '✨ Running Black code formatter check...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    black --check --config .black .
                '''
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Running unittest...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    echo "Running tests with unittest..."
                    python -m unittest discover -s tests -p "test*.py" -v
                '''
            }
            post {
                always {
                    junit testResults: '**/test-results/*.xml', allowEmptyResults: true
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo '🚀 Deploying Python application...'
                sh '''
                    . ${VENV_DIR}/bin/activate || true
                    echo "Running the calculator application..."
                    python -m src.calculator || echo "No direct entry point found"
                '''
                echo '✅ Deployment stage completed'
            }
        }
    }
    
    post {
        always {
            echo '🧹 Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo '🎉 Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}