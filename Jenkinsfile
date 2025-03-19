pipeline {
  agent any
  environment {
    LAST_GOOD_COMMIT = ""
    BAD_COMMIT = ""
    TEST_FAILED = "false"
  }
  stages {
    stage('check out') {
      steps {
        git(url: 'https://github.com/SeanBakker/ece453-maven-samples.git', branch: 'master')

        script {
          // Retrieve last good commit from previous build description
          def lastDescription = currentBuild.rawBuild.getPreviousBuild()?.getDescription()
          if (lastDescription && lastDescription.startsWith("Last Good Commit:")) {
            env.LAST_GOOD_COMMIT = lastDescription.split(":")[1].trim()
          }
        }
      }
    }

    stage('run') {
      steps {
        script {
          try {
            bat 'mvn clean test'

            // If tests pass, store LAST_GOOD_COMMIT as the latest commit hash
            def currentCommit = bat(script: 'git rev-parse HEAD', returnStdout: true).trim()
            env.LAST_GOOD_COMMIT = currentCommit

            // Persist the value by updating the build description
            currentBuild.description = "Last Good Commit: ${env.LAST_GOOD_COMMIT}"

          } catch (Exception e) {
            // If tests fail, mark the current commit as bad
            env.BAD_COMMIT = bat(script: 'git rev-parse HEAD', returnStdout: true).trim()
            env.TEST_FAILED = "true"
          }
        }
      }
    }

    stage('git bisect (if tests fail)') {
      when {
        expression { env.TEST_FAILED == "true" && env.LAST_GOOD_COMMIT != "" }
      }
      steps {
        script {
          bat """
          echo Running git bisect between %LAST_GOOD_COMMIT% (good) and %BAD_COMMIT% (bad)
          git bisect start %BAD_COMMIT% %LAST_GOOD_COMMIT%
          git bisect run mvn clean test
          git bisect reset
          """
        }
      }
    }
  }
  tools {
    maven 'DHT_MVN'
    jdk 'DHT_SENSE'
  }
}
