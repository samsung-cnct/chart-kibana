pipeline {
  agent any
  stages {
    stage('test') {
      steps {
        sh 'build/test.sh'
      }
    }
    stage('stage') {
      steps {
        sh 'build/build.sh'
      }
    }
  }
}