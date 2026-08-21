pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins-agent: vprofile
spec:
  containers:
  - name: maven
    image: maven:3.9.6-eclipse-temurin-11
    command:
    - cat
    tty: true
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - cat
    tty: true
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
'''
        }
    }

    environment {
        REGISTRY = "192.168.56.35:5000"
        IMAGE_NAME = "vprofileapp"
        MAVEN_OPTS = "-Xmx384m -Xms128m -XX:+UseSerialGC"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
