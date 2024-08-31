A jenkins pipeline which converts a Google Sheet tab into a CSV and publishes it to a git repo.

Use this in the following scenario
-----

* You have a Google Sheets spreadsheet such as https://docs.google.com/spreadsheets/d/abc123
* Your Google Sheet is publicly available: you can visit it in an incognito window and see it
* Your Google Sheet has a sheet called "mySheet"
* You would like the contents of mySheet to be exported to a CSV and added to a git repo at path/to/csv.csv
* You have a Jenkins server

How to use this
-----

### Make the main pipeline job, which is parametrized

* Create a Jenkins job called public-google-sheet-to-csv-in-repo
* In pipeline, select "Pipeline script from SCM", and use the master branch of `https://github.com/dcycle/jenkins-pipeline-public-google-sheet-to-csv-in-repo.git`
* Make the project parametrized, with the following string parameters: GOOGLE_SHEETS_SPREADSHEET_ID (which in this example will be abc123) and GOOGLE_SHEETS_SHEET_ID (which in this example is mySheet)

### Make a trigger job, which is built periodically and not parametrized

* Create a Jenkins job called trigger-public-google-sheet-to-csv-in-repo-XYZ, where XYZ is the description of the sheet you want to publish
* Build it periodically (for example `@daily`) because we currently cannot be notified when the Google sheet is modified
