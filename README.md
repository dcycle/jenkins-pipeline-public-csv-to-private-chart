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

### Make sure you have a git repo which can receive the file

This repo will be, for example git@github.com:alberto56/beh-indicateurs.git

Next we need a deploy key pair. On your local computer, run:

    ssh-keygen -t ed25519 -C "your_email@example.com"

When it asks you the location of the key pair, put a temporary location such as:

    /Users/me/.ssh/temp

Do not use a passphrase.

Now, /Users/me/.ssh/temp.pub will contain the public key, and /Users/me/.ssh/temp will contain the private key.

In the GitHub interface for the repo, go to settings > deploy keys, and add a new deploy key with the title Jenkins, the contents of your public key, and allow write access.

In Jenkins credentials, add a credential of type "Secret file", call it MY_REPO_DEPLOY_KEY; put the username git, and add your private key (you might first have to copy it to your ~/Desktop folder or the file manager won't see it because .ssh is hidden).

### Create the google-sheets-to-csv-public job

Parametrize it with:

* GOOGLE_SHEETS_SHEET_ID (for example mySheet)
* GOOGLE_SHEETS_SPREADSHEET_ID (for example abc123)
* CSV_FILE_LOCATION (for example /var/jenkins_home/workspace/google-sheets-to-csv-public/export.csv)
* CSV_FILENAME (for example export.csv)

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
       "$CSV_FILENAME"

This will get the file as "export.csv" (*not* data.csv!) as export.csv is hard-coded in ./scripts/jenkins/fetch-google-sheets.sh.

In Post-build actions, select "Archive the artifacts" and enter "*.csv".

### Create the job which pushes a file to a repo

* Create a Jenkins job called publish-file-to-repo
* In Source code management, select git and https://github.com/dcycle/publish-file-to-repo.git
* Select "This project is parameterized" and add the following string parameters:
  * GIT_REPO (for example git@github.com:alberto56/beh-indicateurs.git)
  * FILE_LOCATION (for example /var/jenkins_home/workspace/google-sheets-to-csv-public/export.csv)
  * MY_REPO_DEPLOY_KEY (a location to a private key) (note that this is a string, not a file)
  * MY_REPO_LOCATION (for example ./path/to/file/in/repo)
* In Build steps, select Execute shell and put `./scripts/publish-file-to-repo.sh`.

### Make the main pipeline job, which is parametrized

* Create a Jenkins job called public-google-sheet-to-csv-in-repo
* In pipeline, select "Pipeline script from SCM", and use the master branch of `https://github.com/dcycle/jenkins-pipeline-public-google-sheet-to-csv-in-repo.git`
* Make the project parametrized, with the following string parameters:
  * GOOGLE_SHEETS_SPREADSHEET_ID (for example abc123)
  * GOOGLE_SHEETS_SHEET_ID (for example mySheet)
  * MY_REPO_DEPLOY_KEY (a location to a private key) (note that this is a string, not a file)
  * MY_REPO (for example git@github.com:alberto56/beh-indicateurs.git)
  * MY_REPO_LOCATION (for example ./path/to/file/in/repo)

### Make a trigger job, which is built periodically and not parametrized

* Create a Jenkins job called trigger-public-google-sheet-to-csv-in-repo-XYZ, where XYZ is the description of the sheet you want to publish
* Build it periodically (for example `@daily`) because we currently cannot be notified when the Google sheet is modified
* Use secret text(s) or file(s), and add secret file MY_REPO_DEPLOY_KEY with the file you entered previously
* In post-build actions, choose "Trigger parameterized build on other projects", and build "public-google-sheet-to-csv-in-repo"
* Use Predefined parameters:

```
GOOGLE_SHEETS_SPREADSHEET_ID=abc123
GOOGLE_SHEETS_SHEET_ID=mySheet
CSV_FILE_LOCATION=/var/jenkins_home/workspace/google-sheets-to-csv-public/export.csv
CSV_FILENAME=export.csv
MY_REPO_DEPLOY_KEY=$MY_REPO_DEPLOY_KEY
MY_REPO=git@github.com:alberto56/beh-indicateurs.git
MY_REPO_LOCATION=./path/to/file/in/repo
```

Linting
-----

You can lint the Jenkinsfile herein locally using:

    npm install -g npm-groovy-lint
    npm-groovy-lint Jenkinsfile
