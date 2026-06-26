pipeline {
    agent any

    parameters {
        string(
            name: 'APP_HOST',
            defaultValue: '',
            description: 'App EC2 private IP (leave empty to skip deploy)'
        )
    }

    environment {
        PATH = "${env.HOME}/.local/bin:${env.PATH}"
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
                sh '''
                    export PATH="$HOME/.local/bin:$PATH"
                    uv sync --frozen
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    export PATH="$HOME/.local/bin:$PATH"
                    uv run python -c "from main import app; print(app.title)"
                '''
            }
        }

        stage('Deploy') {
            when {
                expression { return params.APP_HOST?.trim() }
            }
            steps {
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh """
                        set -e
                        APP_HOST="${params.APP_HOST.trim()}"

                        rsync -avz --delete \\
                          -e "ssh -o StrictHostKeyChecking=no" \\
                          --exclude .venv --exclude .git \\
                          ./ ubuntu@\${APP_HOST}:/opt/hello-api/

                        ssh -o StrictHostKeyChecking=no ubuntu@\${APP_HOST} 'bash -s' <<'EOF'
export PATH="\$HOME/.local/bin:\$PATH"
if ! command -v uv >/dev/null 2>&1; then
    curl -LsSf https://astral.sh/uv/install.sh | sh
    export PATH="\$HOME/.local/bin:\$PATH"
fi
cd /opt/hello-api
uv python install 3.13
uv sync --frozen
sudo systemctl restart hello-api
EOF
                    """
                }
            }
        }

        stage('Health Check') {
            when {
                expression { return params.APP_HOST?.trim() }
            }
            steps {
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${params.APP_HOST.trim()} \\
                          "curl -sf http://localhost:8000/ | grep hello"
                    """
                }
            }
        }
    }
}
