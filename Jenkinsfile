
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE = 'devops-2048:1.0'
        CLUSTER = 'devops-lab'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Source') {
            steps {
                script {
                    if (!fileExists('app/index.html') ||
                        !fileExists('app/style.css') ||
                        !fileExists('app/game.js') ||
                        !fileExists('Dockerfile')) {
                        error('Required project files are missing')
                    }
                }
                echo 'Required source files exist'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE% .'
            }
        }

        stage('Test Container') {
            steps {
                bat '''
                    docker rm -f game2048-ci 2>nul
                    docker run -d --name game2048-ci -p 18080:80 %IMAGE%
                    powershell -NoProfile -Command "$ok=$false; for($i=0;$i -lt 15;$i++){ try { $r=Invoke-WebRequest http://localhost:18080/health -UseBasicParsing -TimeoutSec 2; if($r.StatusCode -eq 200 -and $r.Content -match 'healthy'){$ok=$true; break} } catch {}; Start-Sleep -Seconds 2 }; if(-not $ok){ docker logs game2048-ci; exit 1 }"
                '''
            }
        }

        stage('Load Image into kind') {
            steps {
                bat 'kind load docker-image %IMAGE% --name %CLUSTER%'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f k8s/deployment.yaml'
                bat 'kubectl apply -f k8s/service.yaml'
                bat 'kubectl rollout status deployment/game2048 --timeout=120s'
            }
        }
    }

    post {
        always {
            bat 'docker rm -f game2048-ci 2>nul'
        }
    }
}
