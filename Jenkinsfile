pipeline {
    agent any

    environment {
        APP_NAME = "my-app"
        JAR_FILE = "target/my-app-1.0-SNAPSHOT.jar"
        PORT = "8080"
        GIT_REPO = "https://github.com/12345dee/jenkins2.0monday.git"
        APP_SERVER = "3.235.30.163" // if deploying remotely
        SSH_USER = "ubuntu"
    }

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {

        stage('Clone Code') {
            steps {
                echo "Cloning repository..."
                git branch: 'master', url: "${GIT_REPO}"
            }
        }

        stage('Build JAR') {
            steps {
                echo "Building Maven package..."
                sh 'mvn clean package'
            }
        }

        stage('Stop Old App') {
            steps {
                echo "Stopping old app..."
                // If deploying on the same Jenkins server
                sh '''
                PID=$(lsof -t -i:${PORT}) || true
                if [ ! -z "$PID" ]; then
                    echo "Stopping running app (PID $PID)"
                    kill -9 $PID
                else
                    echo "No app running on port ${PORT}"
                fi
                '''
            }
        }

        stage('Deploy New App') {
            steps {
                echo "Deploying new JAR..."
                sh '''
                nohup java -jar ${JAR_FILE} > app.log 2>&1 &
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Verifying deployment..."
                script {
                    sleep 5
                    sh '''
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:${PORT})
                    if [ "$STATUS" != "200" ]; then
                        echo "Deployment failed!"
                        exit 1
                    else
                        echo "Deployment successful!"
                    fi
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs!"
        }
        always {
            cleanWs()
        }
    }
}
