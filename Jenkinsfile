pipeline {
    agent any

    // this section configures Jenkins options
    options {

        // only keep 10 logs for no more than 10 days
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))

        // cause the build to time out if it runs for more than 12 hours
        timeout(time: 12, unit: 'HOURS')
    }

    // this section configures triggers
    triggers {
          // Poll SCM every 5 minutes to check for changes
          // Format: minute hour day month dayOfWeek
          // H/5 * * * * = every 5 minutes
          pollSCM('H/5 * * * *')
    }

    // the pipeline section we all know and love: stages! :D
    stages {
        stage('Requirements') {
            steps {
                timestamps {
                    echo 'Installing requirements...'
                }
            }
        }
        stage('Build') {
            steps {
                timestamps {
                    echo 'Building..'
                }
            }
        }
        stage('Test') {
            steps {
                timestamps {
                    echo 'Testing..'
                }
            }
        }
        stage('Report') {
            steps {
                timestamps {
                    echo 'Reporting....'
                }
            }
        }
    }

    // the post section is a special collection of stages
    // that are run after all other stages have completed
    post {

        // the always stage will always be run
        always {

            // the always stage can contain build steps like other stages
            // a "steps{...}" section is not needed.
            echo "This step will run after all other steps have finished.  It will always run, even in the status of the build is not SUCCESS"
        }
    }
}