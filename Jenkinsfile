import hudson.tasks.test.AbstractTestResultAction
def jobnameparts = JOB_NAME.tokenize('/') as String[]
def jobconsolename = jobnameparts[0]
def summary = ""
pipeline {

    agent {
        node {
            label 'master'
            customWorkspace "${jobconsolename}"
        }
    }

    tools {
        jdk 'Java 8'
        maven 'Maven'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '2'))
    }

    stages {
        stage('Example Build') {
            steps {
                sh 'echo $JAVA_HOME'
                sh 'mvn clean install'
            }
        }
        stage('Example Test') {
            steps {
                echo 'Testing'
            }
        }
        stage('Build pipleline sub') {
            steps {
                dir('pipeline-sub') {
                    git branch: 'mybranch',
                            url: 'https://github.com/prasadlvi/pipeline-sub.git'
                    sh '''
                       cat README.md
                      '''
                }
                sh '''
                   export prev_version='My previous version'
                   cd ..
                   export branch_name=${PWD##*/}
                   cd -
                   ./Build/ova-build.sh
                '''
               
            }
        }
    }

    post {
        always {
            junit 'target/surefire-reports/TEST-*.xml'
            archiveArtifacts artifacts: 'target/*'

            script {
                if (currentBuild.result == null) {
                    currentBuild.result = 'SUCCESS'
                } else if(currentBuild.result == 'FAILURE' || currentBuild.result == 'UNSTABLE') {
                    emailext to: 'prasad@lvi.co.jp', subject: '$DEFAULT_SUBJECT', body: '$DEFAULT_CONTENT'
                    echo "Failure unstable email sent"
                }
                def testResultAction = currentBuild.rawBuild.getAction(AbstractTestResultAction.class)
                if (testResultAction != null) {
                    def total = testResultAction.getTotalCount()
                    def failed = testResultAction.getFailCount()
                    def skipped = testResultAction.getSkipCount()

                    summary = "\nTest results:\n\t"
                    summary = summary + ("Passed: " + (total - failed - skipped))
                    summary = summary + (", Failed: " + failed)
                    summary = summary + (", Skipped: " + skipped)
                } else {
                    summary = "No tests found"
                }
            }

           // mattermostSend message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} after ${currentBuild.durationString.replace(' and counting', '')} <${env.BUILD_URL}|Open>${summary}"
            echo "${summary}"
            // echo "Test Status:\n  Passed: ${passed}, Failed: ${failed} ${testResultAction.failureDiffString}, Skipped: ${skipped}"

            echo "RESULT: ${currentBuild.result}"
            echo "Duration : ${currentBuild.durationString.replace(' and counting', '')}"

        }
    }

}
