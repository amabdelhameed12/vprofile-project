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
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "1536Mi"
        cpu: "1000m"
    volumeMounts:
    - name: m2-cache
      mountPath: /root/.m2
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - cat
    tty: true
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "1024Mi"
        cpu: "1000m"
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: m2-cache
    hostPath:
      path: /tmp/m2-cache
      type: DirectoryOrCreate
  - name: docker-config
    secret:
      secretName: dockerhub-secret
      items:
      - key: .dockerconfigjson
        path: config.json
'''
        }
    }

    environment {
        DOCKERHUB_USER = "ahmed17793"
        IMAGE_NAME = "vprofileapp"
        MAVEN_OPTS = "-Xmx1024m -Xms256m -XX:+UseG1GC"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']],
                    extensions: [[$class: 'CloneOption', depth: 1, noTags: true, shallow: true, timeout: 15]],
                    userRemoteConfigs: [[url: 'https://github.com/amabdelhameed12/vprofile-project.git']]
                ])
            }
        }

        stage('Test & Build WAR') {
            steps {
                container('maven') {
                    sh 'mvn clean package -DskipTests --batch-mode'
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
                                     --destination=${DOCKERHUB_USER}/${IMAGE_NAME}:${env.BUILD_NUMBER} \
                                     --destination=${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI Pipeline Completed Successfully. Docker Image pushed to Docker Hub."
        }
        failure {
            echo "CI Pipeline Failed."
        }
    }
}
