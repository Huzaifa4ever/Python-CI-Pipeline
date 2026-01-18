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
                source $VENV/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        // -------------------------
        stage('Lint') {
            steps {
                echo "Running flake8 lint check..."
                sh '''
                source $VENV/bin/activate
                flake8 app tests
                '''
            }
        }

        // -------------------------
        stage('Test') {
            steps {
                echo "Running pytest with JUnit report..."
                sh '''
                mkdir -p reports
                source $VENV/bin/activate
                pytest --junitxml=reports/junit.xml --cov=app
                '''
            }
            post {
                always {
                    junit 'reports/junit.xml'
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
