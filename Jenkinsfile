pipeline {
  agent {
    docker {
      image 'akamai/shell'
    }

  }
  stages {
    stage('Find Property') {
      steps {
        sh 'akamai --help'
      }
    }

  }
}