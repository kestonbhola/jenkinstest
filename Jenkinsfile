pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        IMAGE_NAME     = "jenkinstest:latest"
        CONTAINER_NAME = "jenkinstest"
        HOST_PORT      = "8081"
        CONTAINER_PORT = "80"
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/kestonbhola/jenkinstest.git/', branch: 'main'
            }
        }

        stage('Verify Project Files') {
            steps {
                sh '''
                    set -e
                    echo "Checking required project files..."

                    test -f Dockerfile
                    test -f nginx.conf
                    test -f index.html
                    test -f sgustyle.css
                    test -f sguscript.js

                    echo "Required files found."
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e
                    docker build --pull -t "$IMAGE_NAME" .
                '''
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    set +e
                    docker rm -f "$CONTAINER_NAME"
                    true
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    set -e
                    docker run -d \
                      --name "$CONTAINER_NAME" \
                      --restart unless-stopped \
                      -p "$HOST_PORT:$CONTAINER_PORT" \
                      "$IMAGE_NAME"
                '''
            }
        }

        stage('Test Website Locally') {
            steps {
                sh '''
                    set -e
                    sleep 2
                    curl -I http://localhost:$HOST_PORT
                '''
            }
        }

        stage('Show Running Container') {
            steps {
                sh '''
                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful.'
            echo 'Open your EC2 public IP followed by :8080 in a browser to view the site.'
        }
        failure {
            echo 'Deployment failed. Check the Jenkins console output.'
        }
    }
}
