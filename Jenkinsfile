pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: jenkins-agent
spec:
  containers:
  - name: maven
    image: maven:3.9.6-eclipse-temurin-11
    command:
    - cat
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - cat
    tty: true
'''
        }
    }

    environment {
        REGISTRY = "192.168.56.35:5000"
        IMAGE_NAME = "vprofileapp"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scss
            }
        }

        stage('Test & Build WAR') {
            steps {
                container('maven') {
                    sh 'mvn clean install -DskipTests'
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.war', allowEmptyArchive: true
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor --context=dir:///home/jenkins/agent/workspace/${env.JOB_NAME} \
                                     --dockerfile=Dockerfile \
                                     --destination=${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER} \
                                     --destination=${REGISTRY}/${IMAGE_NAME}:latest \
                                     --insecure \
                                     --skip-tls-verify
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI Pipeline Completed Successfully. Artifacts and Docker Image pushed."
        }
        failure {
            echo "CI Pipeline Failed."
        }
    }
}
