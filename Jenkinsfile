pipeline {
  agent any
  stages {
    stage('check out') {
      steps {
        git(url: 'https://github.com/SeanBakker/ece453-maven-samples.git', branch: 'master')
      }
    }

    stage('run') {
      agent any
      steps {
        bat 'mvn clean'
        bat 'mvn test'
        bat 'mvn verify'
      }
    }

  }
  tools {
    maven 'DHT_MVN'
    jdk 'DHT_SENSE'
  }
}
