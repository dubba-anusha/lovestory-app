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
        DOCKERHUB_USERNAME = 'anushadubba'
        IMAGE_TAG = "${BUILD_NUMBER}"
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

        stage('Deploy WAR to Tomcat') {
            steps {
                sh '''
                APP_CONTEXT="lovestory-${DEPLOY_ENV}"

                curl -u $TOMCAT_USER:$TOMCAT_PASS \
                  -T target/lovestory.war \
                  "$TOMCAT_URL/manager/text/deploy?path=/${APP_CONTEXT}&update=true"
                '''
            }
        }

        stage('Build Docker Image with Kaniko') {
            steps {
                sh '''
                cat > /tmp/kaniko-job.yaml <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: kaniko-lovestory-build-${BUILD_NUMBER}
spec:
  backoffLimit: 0
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: kaniko
          image: gcr.io/kaniko-project/executor:latest
          args:
            - "--context=git://github.com/dubba-anusha/lovestory-app.git"
            - "--dockerfile=Dockerfile"
            - "--destination=docker.io/anushadubba/lovestory-app:${BUILD_NUMBER}"
          volumeMounts:
            - name: docker-config
              mountPath: /kaniko/.docker
      volumes:
        - name: docker-config
          secret:
            secretName: dockerhub-secret
            items:
              - key: .dockerconfigjson
                path: config.json
EOF

                /tmp/kubectl apply -f /tmp/kaniko-job.yaml
                /tmp/kubectl wait --for=condition=complete job/kaniko-lovestory-build-${BUILD_NUMBER} --timeout=300s
                /tmp/kubectl logs job/kaniko-lovestory-build-${BUILD_NUMBER}
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
