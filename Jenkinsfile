pipeline {
    agent any

    environment {
        VENV = "${WORKSPACE}/venv"
    }

    stages {

        // -------------------------
        stage('Setup') {
            steps {
                echo "Creating virtual environment and installing dependencies..."
                sh '''
                python3 -m venv $VENV
                $VENV/bin/pip install --upgrade pip
                $VENV/bin/pip install -r requirements.txt
                '''
            }
        }

        // -------------------------
        stage('Lint') {
            steps {
                echo "Running flake8 lint check..."
                sh '''
                $VENV/bin/flake8 app tests
                '''
            }
        }

        // -------------------------
        stage('Test') {
            steps {
                echo "Running tests with pytest..."
                sh '''
                mkdir -p reports
                $VENV/bin/pytest --junitxml=reports/junit.xml --cov=app --cov-report=html:reports/htmlcov
                '''
            }
            post {
                always {
                    junit 'reports/junit.xml'

                    publishHTML(target: [
                        reportName: 'Coverage Report',
                        reportDir: 'reports/htmlcov',
                        reportFiles: 'index.html',
                        keepAll: true,
                        alwaysLinkToLastBuild: true
                    ])
                }
            }
        }

        // -------------------------
        stage('Build') {
            steps {
                echo "Packaging the app..."
                sh '''
                mkdir -p build
                zip -r build/app.zip app
                '''
                archiveArtifacts artifacts: 'build/app.zip', fingerprint: true
            }
        }

        // -------------------------
        stage('Deploy') {
            steps {
                echo "Deploying artifact to server folder..."
                sh '''
                mkdir -p server
                cp build/app.zip server/
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
