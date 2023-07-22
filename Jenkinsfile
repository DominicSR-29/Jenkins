import groovy.time.TimeCategory
import java.text.SimpleDateFormat
def currentDate(){
    date = new Date() 
    sdf = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'") 
    return sdf.format(date)
}
def before1day(){
    currentDate =  new Date()  
    use( TimeCategory ) { 
        int timeframe = java.lang.Integer.parseInt("1");
        before1day = currentDate - timeframe
    }
    sdf = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'")
    return sdf.format(before1day)
}
def getLogGroups() {
    LOG_GROUP = sh (
    script: "aws logs describe-log-groups --region us-east-1 --output text|awk '{print \$4}'",
    returnStdout: true
    ).trim().toString()
    //values = UUID.randomUUID().toString().split('-').join('\n')
    return LOG_GROUP
}
def getLogStream(log) {
    LOG_STREAM = sh (
    script: "aws logs describe-log-streams --log-group-name ${log} --region us-east-1 --output text |awk '{print \$7}'",
    returnStdout: true
    ).trim().toString()
    //values = UUID.randomUUID().toString().split('-').join('\n')
    return LOG_STREAM
}
pipeline {
    agent any
    environment {
            MY_LOG_STREAM = "None"
    }
   
    stages {
        stage("Choose LogGroup") {
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
        stage("Choose LogStream") {
            when {
                expression {
                    return env.LOG_STREAM_FLAG == 'true';
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
        stage("List Logs") {
         steps {
                sh "python3.7 /var/jenkins_home/log.py ${env.LOG_GROUP} --start=${env.START_TIME} --end=${env.END_TIME} --log-stream-names=${MY_LOG_STREAM}  --filter-pattern=${env.PATTERN} > generatedLog.txt"
                echo "\n\n\n+++++++++++++++++++++++++++++++++++++++++++++++"
                echo "+++++++++++++++++++++++++++++++++++++++++++++++++"
                echo "+++++++++++++++++++++++++++++++++++++++++++++++++"
                echo "Sample Log"
                sh "tail -n 30 generatedLog.txt"
                echo "\n\n\n+++++++++++++++++++++++++++++++++++++++++++++++"
                echo "+++++++++++++++++++++++++++++++++++++++++++++++++"
                echo "+++++++++++++++++++++++++++++++++++++++++++++++++"
                // sh "zip generatedLog.zip generatedLog.txt"
                archiveArtifacts artifacts: 'generatedLog.txt', onlyIfSuccessful: true
         }
        }
    }
}
