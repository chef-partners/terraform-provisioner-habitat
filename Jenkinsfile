def pod_label = "habprov-tf-${UUID.randomUUID().toString()}"
pipeline {
  agent {
    kubernetes {
      label pod_label
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: habprov
    image: gcr.io/spaterson-project/jenkins-habtfprov:latest
    command: ['cat']
    tty: true
    alwaysPullImage: true
"""
    }
  }
  stages {
    stage('Build Information') {
        steps {
            container('habprov') {
                sh 'pwd'
                sh 'ls -al'
                sh 'echo PATH = $PATH'
                sh 'git --version'
                sh 'chmod +x ./build.sh'
                sh 'chmod +x ./test.sh'
                dir ('/home/jenkins/workspace/TF-Hab-Provisioner_master') { 
                  sh('bash build.sh')
                }  
              }
            }
    }
    stage('Test TF Habitat Provisioner') {
        steps {
            container('habprov') {
                dir ('/home/jenkins/workspace/TF-Hab-Provisioner_master') { 
                  sh('bash test.sh')
                }  
            }
        }
    }
  }
  triggers {
    cron 'H 10 * * *'
  }
  post {
    success {
        slackSend color: 'good', message: "The pipeline ${currentBuild.fullDisplayName} completed successfully. <${env.BUILD_URL}|Details here>."
    }
    failure {
        slackSend color: 'danger', message: "Pipeline failure ${currentBuild.fullDisplayName}. Please <${env.BUILD_URL}|resolve issues here>."
    }
  }
  options {
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
  }
}
