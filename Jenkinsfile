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

        stage('Setup') {
            steps {
                sh '''
                    export PATH="$HOME/.local/bin:$PATH"
                    if ! command -v uv >/dev/null 2>&1; then
                        curl -LsSf https://astral.sh/uv/install.sh | sh
                        export PATH="$HOME/.local/bin:$PATH"
                    fi
                    uv --version
                    uv python install 3.13
                '''
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
