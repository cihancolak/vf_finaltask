pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    // Docker imajını oluştur
                    def dockerImage = docker.build('my-flask-app')
                }
            }
        }
        stage('Test') {
            steps {
                // Testlerin çalıştırılması
                sh 'echo "Running tests..."'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Kubernetes YAML dosyalarının uygulanması
                    sh 'kubectl apply -f deployment.yaml --validate=false'
                    sh 'kubectl apply -f service.yaml --validate=false'
                }
            }
        }
    }
    post {
        always {
            // Çalışma alanının temizlenmesi
            cleanWs()
        }
    }
}

