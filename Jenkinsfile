pipeline {
    agent any

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'qa', 'prod'],
            description: 'Choose where to deploy'
        )
    }

    environment {
        TOMCAT_URL = 'http://host.minikube.internal:8082'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin123'
    }

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
                APP_CONTEXT="lovestory-${DEPLOY_ENV}"

                echo "Deploying to context: ${APP_CONTEXT}"

                curl -u $TOMCAT_USER:$TOMCAT_PASS \
                  -T target/lovestory.war \
                  "$TOMCAT_URL/manager/text/deploy?path=/${APP_CONTEXT}&update=true"
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
