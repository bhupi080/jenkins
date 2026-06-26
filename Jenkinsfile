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
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh """
                        set -e
                        APP_HOST="${params.APP_HOST.trim()}"
                        chmod 600 \$SSH_KEY

                        rsync -avz --delete \\
                          -e "ssh -i \$SSH_KEY -o StrictHostKeyChecking=no" \\
                          --exclude .venv --exclude .git \\
                          ./ \${SSH_USER}@\${APP_HOST}:/opt/hello-api/

                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \${SSH_USER}@\${APP_HOST} 'bash -s' <<'EOF'
export PATH="\$HOME/.local/bin:\$PATH"
if ! command -v uv >/dev/null 2>&1; then
    curl -LsSf https://astral.sh/uv/install.sh | sh
    export PATH="\$HOME/.local/bin:\$PATH"
fi
cd /opt/hello-api
uv python install 3.13
uv sync --frozen
sudo cp deploy/hello-api.service /etc/systemd/system/hello-api.service
sudo systemctl daemon-reload
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
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh """
                        chmod 600 \$SSH_KEY
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \${SSH_USER}@${params.APP_HOST.trim()} \\
                          "curl -sf http://localhost:9000/ | grep hello"
                    """
                }
            }
        }
    }
}
