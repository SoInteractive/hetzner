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
    GIT_URL = sh( script: "git config --get remote.origin.url", returnStdout: true ).trim()
    CHANGE_ID = env.BRANCH_NAME.replaceFirst(/^PR-/, "")
    REPO_NAME = sh ( script: "basename `git rev-parse --show-toplevel`", returnStdout: true).trim()
  }
  stages {
    stage('Show variables') {
      steps {
        sh 'env | sort'
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
  stage('Merge Pull Request'){
      when { branch "PR-*" }
      steps {
          withCredentials([[$class: 'StringBinding', credentialsId: '84b13c41-cc5e-4802-b057-e85c232d347b', variable: 'ACCESS_TOKEN_PASSWORD']]) {
                    sh "curl -X PUT -d '{\"commit_title\": \"Merge pull request\"}' https://api.github.com/repos/SoInteractive/$REPO_NAME/pulls/$CHANGE_ID/merge?access_token=$ACCESS_TOKEN_PASSWORD"
        }
      }
    }
/*  stage('Import to ansible galaxy'){
      when { branch "PR-*" }
      steps {
        withCredentials([[$class: 'StringBinding', credentialsId: '84b13c41-cc5e-4802-b057-e85c232d347b', variable: 'GITHUB_TOKEN']]) {
          sh 'ansible-galaxy login --github-token $GITHUB_TOKEN'
          sh 'ansible-galaxy import SoInteractive ${JOB_NAME.split('/')[1]}'
        }
      }
    }*/
  }

  post {
    always {
      sh 'molecule destroy'
    }
    success {
      mattermostSend color: 'good', message: "Pipeline <${RUN_DISPLAY_URL}|#${BUILD_NUMBER}> of <https://github.com/SoInteractive/${JOB_NAME.split('/')[1]}/tree/${BRANCH_NAME}|${JOB_NAME}> branch by ${GIT_COMMITER} finished successfully in ${currentBuild.durationString.replaceAll('and counting','')}"
    }
    failure {
      mattermostSend color: 'danger', message: "Pipeline <${RUN_DISPLAY_URL}|#${BUILD_NUMBER}> of <https://github.com/SoInteractive/${JOB_NAME.split('/')[1]}/tree/${BRANCH_NAME}|${JOB_NAME}> branch by ${GIT_COMMITER} failed in ${currentBuild.durationString.replaceAll('and counting','')}"
    }
  }
}
