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
        stage('Publishes the CSV to a repo') {
            steps {
                build job: 'publish-file-to-repo', parameters: [
                    string(
                        // Yes, I know, I can stash/unstash, but after a few
                        // hours I simply cannot get it to work, so I'm
                        // fetching the CSV_FILE_LOCATION directly.
                        name: 'FILE_LOCATION',
                        value: params.CSV_FILE_LOCATION
                    ),
                    string(
                        name: 'GIT_REPO',
                        value: params.MY_REPO
                    ),
                    string(
                        name: 'MY_REPO_DEPLOY_KEY',
                        value: params.MY_REPO_DEPLOY_KEY
                    ),
                    string(
                        name: 'MY_REPO_LOCATION',
                        value: params.MY_REPO_LOCATION
                    ),
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
    }
}
