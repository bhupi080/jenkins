pipeline {
    agent any

    environment {
        PATH = "/home/jenkins/.local/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'uv sync --frozen'
            }
        }

        stage('Verify') {
            steps {
                sh 'uv run python -c "from main import app; print(app.title)"'
            }
        }
    }
}