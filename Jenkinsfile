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
    stage('Static Analysis') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
        stage('OSS License Checker') {
          steps {
            container('licensefinder') {
              sh 'ls -al'
              sh '''#!/bin/bash --login
                /bin/bash --login
                rvm use default
                gem install license_finder
                license_finder
              '''
            }
          }
        }
        stage('SCA'){
          steps{
            container('maven'){
              catchError(buildResult:'SUCCESS',stageResult:'FAILURE'){
                sh'mvn org.owasp:dependency-check-maven:check'
              }
            }
          }
          post{
            always{
              archiveArtifacts allowEmptyArchive: true, 
              artifacts:'target/dependency-check-report.html',
              fingerprint: true,
              onlyIfSuccessful: true//dependencyCheckPublisherpattern:'report.xml'
            }
          }
        }
      }
    }
    stage('Package') {
      stages {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
              sh 'ls -l target'  // Debug JAR creation
            }
          }
        }
        stage('OCI Image BnP') {
          steps{
            container('kaniko') {
              sh '''
                ls -l /home/jenkins/agent/workspace/dso-demo_main  # Debug workspace
                ls -l /home/jenkins/agent/workspace/dso-demo_main/target  # Debug target dir
                cat /kaniko/.docker/config.json  # Debug credentials
                /kaniko/executor \
                  --dockerfile Dockerfile \
                  --context /home/jenkins/agent/workspace/dso-demo_main \
                  --insecure \
                  --skip-tls-verify \
                  --cache=true \
                  --destination=docker.io/leodocker0808/dso-demo
              '''
            }
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
