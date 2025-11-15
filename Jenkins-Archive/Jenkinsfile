pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['DEVELOPMENT', 'STAGING', 'PRODUCTION'],
            description: 'Select your Environment :'
        )
        password(
            name: 'APIKEY',
            defaultValue: '123ABC',
            description: 'Enter the api key :'
        )

        text(
            name: 'CHANGELOG',
            description: 'Enter your changelog here:',
            defaultValue: 'This is the default changelog.'
        )
    }

    stages {
        stage('Test') {
            when {
                expression { params.ENVIRONMENT != 'PRODUCTION' }
            }           
            steps {
                echo "Running tests for ${params.ENVIRONMENT} environment."
            }
        }
        stage('Deploy') {
            when {
                expression { params.ENVIRONMENT == 'PRODUCTION' }
            }
            steps {
               input message: "Approve deployment to PRODUCTION?", ok: "Deploy"
               echo "Deploying to PRODUCTION environment with API Key: ${params.APIKEY}"
            }
        }
        stage('Report') {
            steps {
               echo "This stage generates a report for PRODUCTION deployment." 
               sh "printf \"Changelog:\n%s\" \"${params.CHANGELOG}\" > \"${params.ENVIRONMENT}_report.txt\""
               archiveArtifacts allowEmptyArchive: true,
               artifacts: "${params.ENVIRONMENT}_report.txt", 
               fingerprint: true,
               followSymlinks: false,
               onlyIfSuccessful: true
            }
        }
    }
}
