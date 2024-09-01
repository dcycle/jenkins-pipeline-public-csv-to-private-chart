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
                        name: 'CSV_FILENAME',
                        value: params.CSV_FILENAME
                    ),
                    string(
                        name: 'GOOGLE_SHEETS_SHEET_ID',
                        value: params.GOOGLE_SHEETS_SHEET_ID
                    )
                ]
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
                // Yes, I know, I can stash/unstash, but after a few hours I
                // simply cannot get it to work, so I'm fetching the
                // CSV_FILE_LOCATION directly.
                sh "cat ${params.CSV_FILE_LOCATION}"
                echo 'Publishes the CSV to a private repo..'
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
        stage('Updates server with new CSV') {
            steps {
                echo 'Updates server with new CSV....'
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
    }
}
