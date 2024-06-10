pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def app = docker.build("my-flask-app")
                }
            }
        }
        stage('Test') {
            steps {
                sh 'echo "Running tests..."'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
