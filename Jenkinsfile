@Library('my-shared-lib') _

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
                buildWar()
            }
        }

        stage('Build and Push Image') {
            steps {
                kanikoBuildPush(env.IMAGE_NAME, params.ENVIRONMENT)
            }
        }
    }
}
