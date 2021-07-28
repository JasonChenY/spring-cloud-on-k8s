pipeline {
    agent {
        kubernetes {
            defaultContainer 'kaniko'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug-v0.19.0
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  - name: maven
    image: maven:3.8.1-openjdk-11-slim
    imagePullPolicy: Always
    command:
    - cat
    tty: true
    volumeMounts:
      - name: volume-0
        mountPath: /root/.m2      
  volumes:
  - name: volume-0
    persistentVolumeClaim:
      claimName: jenkins-agent-pvc-maven
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: docker-credentials
          items:
            - key: .dockerconfigjson
              path: config.json
"""
        }
    }
    environment {
        appName = "springboot-helloworld"
        appVersion = "v2.9.1"
        IMAGE_PUSH_DESTINATION="hkccr.ccs.tencentyun.com/chenyong/${appName}:${appVersion}"
    }
    stages {
        stage("clone") {
            steps {
                git branch: "master", url: "https://github.com/JasonChenY/spring-cloud-on-k8s.git"
            }
        }
        stage("build") {
            steps {
                container("maven") {
                  sh "cd api-gateway && mvn clean package"
                }
            }
        }

        stage('Build with Kaniko') {
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    withEnv(['PATH+EXTRA=/busybox']) {
                        sh '''#!/busybox/sh
                            cp api-gateway/target/*.jar api-gateway/docker-k8s
                            cd api-gateway/docker-k8s
                            /kaniko/executor -f Dockerfile --context `pwd` --destination $IMAGE_PUSH_DESTINATION
                        '''
                    }
                }
            }
        }
    }
}

