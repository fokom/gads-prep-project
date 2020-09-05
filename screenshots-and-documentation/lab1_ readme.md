# Lab 1 Readme

## Console and Cloud Shell

### Step 1 : Get access to Google Cloud.
 From command line(Windows), BASH(linux), run  the following command to initialize gcloud and  press Y to login when prompted. 

 `gcloud init`

### Step 2 : Create a Cloud Storage bucket using the Cloud Console. (command line version)
This command creates a bucket in US multi-region with Standard storage class

`gsutil mb -l US gs://gads-lab-1`

### Step 3 : Create a second Cloud Storage bucket using Cloud Shell.

`gsutil mb gs://gads-lab-test`

### Step 4: copy a file called `credentials.json` from working directory to second bucket
` gsutil cp credentials.json gs://gads-lab-test`



#### Completion of the lab produced the following result from gmail. 
![Image of lab 1](screenshots/console-and-cloud-shell.png)