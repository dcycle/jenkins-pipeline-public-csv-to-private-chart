pipeline {
    agent any

    stages {
        stage('Converts a Google Sheet to CSV') {
            steps {
                build job: 'google-sheets-to-csv-public', parameters: [
                    string(
                        name: 'GOOGLE_SHEETS_SPREADSHEET_ID',
                        value: params.GOOGLE_SHEETS_SPREADSHEET_ID
                    ),
                    string(
                        name: 'GOOGLE_SHEETS_SHEET_ID',
                        value: params.GOOGLE_SHEETS_SHEET_ID
                    )
                ]
                stash includes: '**/target/*.csv', name: 'mystash'
            }
            post {
                failure {
                    script {
                        error 'Failed, exiting now...'
                    }
                }
                unstable {
                    script {
                        error 'Unstable, exiting now...'
                    }
                }
            }
        }
        stage('Publishes the CSV to a private repo') {
            steps {
                unstash 'mystash'
                sh 'ls -lah'
                echo 'Publishes the CSV to a private repo..'
            }
        }
        stage('Updates server with new CSV') {
            steps {
                echo 'Updates server with new CSV....'
            }
        }
    }
}
