pipeline {
  agent any

  stages {
    stage('Converts a Google Sheet to CSV') {
      steps {
        build job: 'google-sheets-to-csv-public', parameters: [
          string(
            name: 'GOOGLE_SHEETS_SHEET_ID',
            params.GOOGLE_SHEETS_SHEET_ID
          ),
          string(
            name: 'GOOGLE_SHEETS_SHEET_ID',
            params.GOOGLE_SHEETS_SHEET_ID
          )
        ]
      }
      post{
        failure {
          script{
            error "Failed, exiting now..."
          }
        }
        unstable {
          script{
            error "Unstable, exiting now..."
          }
        }
      }
    }
    stage('Publishes the CSV to a private repo') {
      steps {
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
