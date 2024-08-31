A jenkins pipeline which converts a Google Sheet tab into a CSV and publishes it to a git repo.

Use this in the following scenario
-----

* You have a Google Sheets spreadsheet such as https://docs.google.com/spreadsheets/d/abc123
* Your Google Sheet is publicly available: you can visit it in an incognito window and see it
* Your Google Sheet has a sheet called "mySheet"
* You would like the contents of mySheet to be exported to a CSV and added to a git repo at path/to/csv.csv
* You have a Jenkins server
* You have a Docker host

How to use this
-----

### In Jenkins credentials, create the API key

Create a secret text called GOOGLE_SHEETS_API_KEY with your Google Sheets API key as defined in https://github.com/dcycle/google-sheets-to-csv.

We are assuming that Jenkins does not have Docker installed and all Docker operations will be on an external docker host. To access that, create the secrets DOCKERHOSTUSER and DOCKERHOST.

### Create the google-sheets-to-csv-public job

Parametrize it with:

* GOOGLE_SHEETS_SHEET_ID (for example mySheet)
* GOOGLE_SHEETS_SPREADSHEET_ID (for example abc123)

In source code management, use the master branch of https://github.com/dcycle/google-sheets-to-csv.git.

In "Use secret text(s) or file(s)", bind:

* GOOGLE_SHEETS_API_KEY
* DOCKERHOSTUSER
* DOCKERHOST

In Build Steps, enter:

    ./scripts/jenkins/fetch-google-sheets.sh \
       "$GOOGLE_SHEETS_API_KEY" \
       "$GOOGLE_SHEETS_SPREADSHEET_ID" \
       "$GOOGLE_SHEETS_SHEET_ID" \
       ./app/unversioned/scripts/data.csv

This will get the file as "export.csv" (*not* data.csv!) as export.csv is hard-coded in ./scripts/jenkins/fetch-google-sheets.sh.

In Post-build actions, select "Archive the artifacts" and choose the file "export.csv".

### Make the main pipeline job, which is parametrized

* Create a Jenkins job called public-google-sheet-to-csv-in-repo
* In pipeline, select "Pipeline script from SCM", and use the master branch of `https://github.com/dcycle/jenkins-pipeline-public-google-sheet-to-csv-in-repo.git`
* Make the project parametrized, with the following string parameters: GOOGLE_SHEETS_SPREADSHEET_ID (which in this example will be abc123) and GOOGLE_SHEETS_SHEET_ID (which in this example is mySheet)

### Make a trigger job, which is built periodically and not parametrized

* Create a Jenkins job called trigger-public-google-sheet-to-csv-in-repo-XYZ, where XYZ is the description of the sheet you want to publish
* Build it periodically (for example `@daily`) because we currently cannot be notified when the Google sheet is modified
* In post-build actions, choose "Trigger parameterized build on other projects", and build "public-google-sheet-to-csv-in-repo"
* Use Predefined parameters:

```
GOOGLE_SHEETS_SPREADSHEET_ID=abc123
GOOGLE_SHEETS_SHEET_ID=mySheet
CSV_FILE_LOCATION=/var/jenkins_home/workspace/google-sheets-to-csv-public/export.csv
```

Linting
-----

You can lint this locally using:

    npm install -g npm-groovy-lint
    npm-groovy-lint Jenkinsfile
