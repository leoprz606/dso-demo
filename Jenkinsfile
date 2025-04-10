pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
      }
    }
    stage('Package') {
      steps {
        stage('Create Jarfile') {
          container('maven') {
            sh 'mvn package -DskipTests'
            sh 'ls -l target'  // Debug JAR creation
          }
        }
        stage('OCI Image BnP') {
          container('kaniko') {
            sh '''
              ls -l /home/jenkins/agent/workspace/dso-demo_main  # Debug workspace
              ls -l /home/jenkins/agent/workspace/dso-demo_main/target  # Debug target dir
              cat /kaniko/.docker/config.json  # Debug credentials
              /kaniko/executor \
                --dockerfile Dockerfile \
                --insecure \
                --skip-tls-verify \
                --cache=true \
                --destination=docker.io/leodocker0808/dso-demo
            '''
          }
        }
      }
    }
    stage('Deploy to Dev') {
      steps {
        // TODO
        sh "echo done"
      }
    }
  }
}
