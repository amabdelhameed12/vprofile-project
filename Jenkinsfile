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
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 8.8.8.8
      - 1.1.1.1
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
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
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
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
  volumes:
  - name: m2-cache
    hostPath:
      path: "/tmp/m2-cache"
      type: "DirectoryOrCreate"
  - name: docker-config
    secret:
      secretName: dockerhub-secret
      items:
      - key: .dockerconfigjson
        path: config.json
  - emptyDir: {}
    name: "workspace-volume"
'''
        }
    }

    environment {
        DOCKERHUB_USER = "ahmed17793"
        IMAGE_NAME = "vprofileapp"
        MAVEN_OPTS = "-Xmx1024m -Xms256m -XX:+UseG1GC"
    }

    stages {
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
