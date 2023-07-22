pipeline {
  agent any
  stages {
    stage('Choose LogGroup') {
      steps {
        timeout(time: 30, unit: 'SECONDS') {
          script {
            def INPUT_PARAMS = input message: 'Please Provide Parameters', ok: 'Next',
            parameters: [
              choice(name: 'LOG_GROUP', choices: getLogGroups(), description: 'Available Log Group'),
              booleanParam(defaultValue: false, description: 'Want to choose log stream', name: 'CHOOSE_STREAM'),
              string(defaultValue: before1day(), description: 'Starting Time', name: 'START_TIME'),
              string(defaultValue: currentDate(), description: 'Ending Time', name: 'END_TIME'),
              string(defaultValue: "error", description: 'Search Pattern', name: 'PATTERN')
            ]
            env.LOG_GROUP = INPUT_PARAMS.LOG_GROUP
            env.START_TIME = INPUT_PARAMS.START_TIME
            env.END_TIME = INPUT_PARAMS.END_TIME
            env.LOG_STREAM_FLAG = INPUT_PARAMS.CHOOSE_STREAM
            env.PATTERN = INPUT_PARAMS.PATTERN
          }

        }

        script {
          echo "FLAG: ${env.LOG_STREAM_FLAG}"
        }

      }
    }

    stage('Choose LogStream') {
      when {
        expression {
          return env.LOG_STREAM_FLAG == 'true'
        }

      }
      steps {
        timeout(time: 30, unit: 'SECONDS') {
          script {
            def userInput = input(
              id: 'userInput', message: 'Enter LogStream?',
              parameters: [
                choice(choices: getLogStream("${env.LOG_GROUP}"),
                description: 'LogStream',
                name: 'MY_LOG_STREAM')
              ])
              MY_LOG_STREAM="${userInput}"
              echo "Log Stream: ${MY_LOG_STREAM}"
            }

          }

        }
      }

      stage('List Logs') {
        steps {
          bat "python3.7 /var/jenkins_home/log.py ${env.LOG_GROUP} --start=${env.START_TIME} --end=${env.END_TIME} --log-stream-names=${MY_LOG_STREAM}  --filter-pattern=${env.PATTERN} > generatedLog.txt"
          echo '''


+++++++++++++++++++++++++++++++++++++++++++++++'''
          echo '+++++++++++++++++++++++++++++++++++++++++++++++++'
          echo '+++++++++++++++++++++++++++++++++++++++++++++++++'
          echo 'Sample Log'
          bat 'tail -n 30 generatedLog.txt'
          echo '''


+++++++++++++++++++++++++++++++++++++++++++++++'''
          echo '+++++++++++++++++++++++++++++++++++++++++++++++++'
          echo '+++++++++++++++++++++++++++++++++++++++++++++++++'
          archiveArtifacts(artifacts: 'generatedLog.txt', onlyIfSuccessful: true)
        }
      }

    }
    environment {
      MY_LOG_STREAM = 'None'
    }
  }