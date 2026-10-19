pipeline {
    agent { label 'docker-agent' }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Code Quality') {
            steps {
                sh '''
                    test -s index.html
                    test -s Dockerfile
                    grep -qi '<html' index.html
                '''
            }
        }

        stage('Build Docker') {
            steps {
                sh 'docker build -t static-website:ci .'
            }
        }

        stage('Tests') {
            steps {
                sh '''
                    docker run --rm static-website:ci nginx -t
                '''
            }
        }

        stage('Acceptance Tests') {
            steps {
                sh '''
                    docker rm -f website-acceptance 2>/dev/null || true
                    docker run -d --name website-acceptance static-website:ci

                    trap 'docker rm -f website-acceptance >/dev/null 2>&1 || true' EXIT

                    docker exec website-acceptance wget -q -O /dev/null http://127.0.0.1/
                    docker exec website-acceptance wget -q -O - http://127.0.0.1/ | grep -qi '<html'
                '''
            }
        }
    }
}
