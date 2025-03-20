pipeline {
  agent any
  stages {
    stage('check out') {
      steps {
        script {
          def scm = checkout([
              $class: 'GitSCM',
              branches: [[name: 'master']],
              doGenerateSubmoduleConfigurations: false,
              extensions: [[$class: 'CloneOption', shallow: false, noTags: false]],
              submoduleCfg: [],
              userRemoteConfigs: [[url: 'https://github.com/SeanBakker/ece453-maven-samples.git']]
          ])
          echo "SCM variables: ${scm}"

          if (GIT_COMMIT_OVERRIDE?.trim()) {
            bat "git checkout %GIT_COMMIT_OVERRIDE%"
          }

          CURRENT_COMMIT = scm['GIT_COMMIT']
          echo "Captured commit: ${CURRENT_COMMIT}"

          def lastBuild = currentBuild.getPreviousBuild()
          if (lastBuild) {
            def lastDescription = lastBuild.getDescription()
            if (lastDescription && lastDescription.startsWith("Last Good Commit:")) {
              LAST_GOOD_COMMIT = lastDescription.split(":")[1].trim()
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
            LAST_GOOD_COMMIT = CURRENT_COMMIT
            echo "Captured commit: ${LAST_GOOD_COMMIT}"

            // Persist the value by updating the build description
            currentBuild.description = "Last Good Commit: ${LAST_GOOD_COMMIT}"

          } catch (Exception e) {
            // Get current git commit
            BAD_COMMIT = CURRENT_COMMIT
            echo "Captured commit: ${BAD_COMMIT}"
            TEST_FAILED = "true"
          }
        }

      }
    }

    stage('git bisect (if tests fail)') {
      when {
        expression {
          TEST_FAILED == "true" && LAST_GOOD_COMMIT != ""
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
    GIT_COMMIT_OVERRIDE = ''
    TEST_FAILED = 'false'
  }
}
