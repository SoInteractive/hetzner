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
  }
  stages {
    stage('Check syntax') {
      steps {
        sh 'env | sort'
        sh 'echo $CHANGE_ID'
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
/*  stage('Merge Pull Request'){
      when { branch "*" }
      steps {
         withCredentials([usernamePassword(credentialsId: 'credential-value', usernameVariable: 'ACCESS_TOKEN_USERNAME', passwordVariable: 'ACCESS_TOKEN_PASSWORD',)]) {
                    sh "curl -X PUT -d '{\"commit_title\": \"Merge pull request\"}'  https://github.ibm.com/api/v3/repos/org-name/repo-name/pulls/$CHANGE_ID/merge?access_token=$ACCESS_TOKEN_PASSWORD"
        }
      }
    }*/
/*  stage('Import to ansible galaxy'){
      when { branch "PR-*" }
      steps {
        withCredentials([[$class: 'StringBinding', credentialsId: 'github-access-token', variable: 'GITHUB_TOKEN']]) {
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
