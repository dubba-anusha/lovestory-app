pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Select environment')
    }

    environment {
        IMAGE_NAME = "anushadubba/lovestory"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build WAR') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('Build and Push Image using Kaniko') {
            steps {
                sh '''
                cat > kaniko-job.yaml <<YAML
apiVersion: batch/v1
kind: Job
metadata:
  name: kaniko-lovestory-${ENVIRONMENT}
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: kaniko
        image: gcr.io/kaniko-project/executor:latest
        args:
        - "--dockerfile=/workspace/Dockerfile"
        - "--context=dir:///workspace"
        - "--destination=docker.io/${IMAGE_NAME}:${ENVIRONMENT}"
        volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker
        - name: workspace
          mountPath: /workspace
      volumes:
      - name: docker-config
        secret:
          secretName: dockerhub-secret
          items:
          - key: .dockerconfigjson
            path: config.json
      - name: workspace
        persistentVolumeClaim:
          claimName: jenkins-pvc
YAML

                /var/jenkins_home/bin/kubectl delete job kaniko-lovestory-${ENVIRONMENT} --ignore-not-found=true
                /var/jenkins_home/bin/kubectl apply -f kaniko-job.yaml
                '''
            }
        }
    }
}
