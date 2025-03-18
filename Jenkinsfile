pipeline {
  agent any
  stages {
    stage('check out') {
      steps {
        git(url: 'https://github.com/SeanBakker/ece453-maven-samples.git', branch: 'master')
      }
    }

    stage('Git Bisect') {
      steps {
        script {
          def badCommit = '198644632661c67b6c32f59e9047c11a70685e15'
          def goodCommit = '98ac319c0cff47b4d39a1a7b61b4e195cfa231e5'
          
          bat '''
            git bisect start ''' + badCommit + ' ' + goodCommit + '''
            git bisect run mvn clean test
            git bisect reset
          '''
        }
      }
    }
  }

  }
  tools {
    maven 'DHT_MVN'
    jdk 'DHT_SENSE'
  }
}
