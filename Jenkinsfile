#!groovy

/* Declarative pipeline */
pipeline {
  agent {
    node {
      label 'master'
      customWorkspace 'workspace/hetzner'
    }
  }
  options {
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '30'))
    timeout(time: 15, unit: 'MINUTES')
  }
  environment {
    GIT_COMMITER = sh( script: "git show -s --pretty=%an", returnStdout: true ).trim()
  }
  stages {
    stage('groovy string manipulation'){
      steps {
        echo "${currentBuild.durationString.replaceAll('and counting','')}"
        echo "${GIT_AUTHOR_EMAIL}"
      }
    }
    stage('Check syntax') {
      steps {
        sh 'molecule syntax'
      }
    }
    stage('Provision environment') {
      steps {
        sh 'molecule create'
      }
    }
    stage('Run ansible') {
      steps {
        sh 'molecule converge'
      }
    }
    stage('Run Tests'){
      steps {
        sh 'molecule idempotence'
        sh 'molecule verify'
      }
    }
/*    stage('Accept code'){
      when { branch "PR-*" }
      steps {
        mergeGithubPullRequest {
          mergeComment('merged by Jenkins')
          disallowOwnCode()
          failOnNonMerge()
          deleteOnMerge()
        }
      }
    }*/
  }

  post {
    always {
      sh 'molecule destroy'
    }
    success {
      mattermostSend color: 'good', message: "Pipeline <${RUN_DISPLAY_URL}|#${BUILD_NUMBER}> of branch ${JOB_NAME} by ${GIT_COMMITER} finished successfully in ${currentBuild.durationString.take(currentBuild.durationString.lastIndexOf('a'))}."
    }
    failure {
      mattermostSend color: 'danger', message: "Pipeline <${RUN_DISPLAY_URL}|#${BUILD_NUMBER}> of branch ${JOB_NAME} by ${GIT_COMMITER_ER} failed in ${currentBuild.durationString.take(currentBuild.durationString.lastIndexOf('a'))}."
    }
  }
}
