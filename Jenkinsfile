pipeline {
  agent any
  stages {
    stage('check out') {
      steps {
        git(url: 'https://github.com/SeanBakker/ece453-maven-samples.git', branch: 'master')
        script {
          if (env.GIT_COMMIT_OVERRIDE?.trim()) {
            bat "git checkout %GIT_COMMIT_OVERRIDE%"
          }

          env.CURRENT_COMMIT = env.GIT_COMMIT
          echo "Captured commit: ${env.CURRENT_COMMIT}"

          def lastBuild = currentBuild.getPreviousBuild()
          if (lastBuild) {
            def lastDescription = lastBuild.getDescription()
            if (lastDescription && lastDescription.startsWith("Last Good Commit:")) {
              env.LAST_GOOD_COMMIT = lastDescription.split(":")[1].trim()
            }
          }
        }

      }
    }

    stage('run') {
      steps {
        script {
          try {
            bat 'mvn clean test'

            // Get current git commit
            env.LAST_GOOD_COMMIT = env.CURRENT_COMMIT
            echo "Captured commit: ${env.LAST_GOOD_COMMIT}"

            // Persist the value by updating the build description
            currentBuild.description = "Last Good Commit: ${env.LAST_GOOD_COMMIT}"

          } catch (Exception e) {
            // Get current git commit
            env.BAD_COMMIT = env.CURRENT_COMMIT
            echo "Captured commit: ${env.BAD_COMMIT}"
            env.TEST_FAILED = "true"
          }
        }

      }
    }

    stage('git bisect (if tests fail)') {
      when {
        expression {
          env.TEST_FAILED == "true" && env.LAST_GOOD_COMMIT != ""
        }

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
  environment {
    CURRENT_COMMIT = ''
    LAST_GOOD_COMMIT = ''
    BAD_COMMIT = ''
    GIT_COMMIT_OVERRIDE = '198644632661c67b6c32f59e9047c11a70685e15'
    TEST_FAILED = 'false'
  }
}