pipeline {
    agent any

    environment {
        APP_NAME = "my-app"
        JAR_FILE = "target/my-app-1.0-SNAPSHOT.jar"
        PORT = "8080"
    }

    options {
        timestamps()
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/YOUR_USERNAME/YOUR_REPO.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Stop Old App') {
            steps {
                script {
                    sh '''
                    PID=$(lsof -t -i:${PORT}) || true
                    if [ ! -z "$PID" ]; then
                        echo "Stopping old app..."
                        kill -9 $PID
                    else
                        echo "No app running"
                    fi
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying new version..."
                nohup java -jar ${JAR_FILE} > app.log 2>&1 &
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
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
