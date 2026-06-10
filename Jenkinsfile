pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                curl -u admin:admin123 \
                  -T target/lovestory.war \
                  "http://host.minikube.internal:8082/manager/text/deploy?path=/lovestory&update=true"
                '''
            }
        }

        stage('Archive WAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }
    }
}
