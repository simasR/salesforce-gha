# Salesfoce CICD using GitHub Actions
This guide will help you easily set up a pipe for your deployments to salesforce environment using GitHub Actions.

In this example I will be using Visual Studio Code as a choice for code editor.

## Preparations
First you will need to install few things that will be used to set everything up.
- Code Editor like [Visual Studio Code (Free)](https://code.visualstudio.com/)
- [SFDX CLI](https://developer.salesforce.com/tools/sfdxcli)
- [Git](https://git-scm.com/downloads)
- [OpenSSL](https://www.openssl.org/) This gets installed with git, and is accessible out of the box through Git Bash. Leaving this here, just in case.

## Setting up a project
### Create a dev org!
Lets create a salesforce [developer org](https://developer.salesforce.com/signup) first.

### Set up a Git repository.
If you dont already have a GitHub account Sign up [here](https://github.com/).
- Create a new repository name it however you like, for example: salesforce-gha.
- Select "Add a README file" checkbox
- Click Create Repository

### Clone git repository.
On your pc find a place where you want your projects to be stored.
- For example, create a folder and name it MyProjects.
- Open that folder
- Right click your mouse
- Select: Git Bash Here.
  - This will open up a Bash Terminal
- Open your browser window where your git repository is
  - click "Code"
  - next to a URL there is copy button, click it.
- Go back to the Bash Terminal you have just opened.
  - type in: git clone <paste that url>  (Don't include < and >)
Now you should have salesforce-gha(If thats how you named your repository) folder within your MyProjects folder.

### Create an sfdx project and Set up your Visual Studio Code (VSC)
- Within your MyProjects **(NOT salesforce-gha!!!)** folder create a file called package.json containing:
```
{
    "name": "yourProjectName"
}
```
- Set up workspace in VSC: 
  - File -> Add folder to workspace...
  - select MyProjects folder

- Set up terminal:
  - Terminal -> New Terminal
  - If Bash terminal was not selected by default I suggest clicking arrow pointing down next to "+" at the top right of the terminal and selecting Git Bash

- Create an sfdx project
  - In terminal run this command:
```
sfdx force:project:create --projectname salesforce-gha --manifest
```
*If project name in this command is same as name of the repository folder, project will be created within your repository root folder, and **that is what we want.***
This will create a bunch of files in your repository root folder. 

- Authorize your org
  - In terminal run this command:
```
sfdx auth:web:login --setalias myDevOrg --instanceurl https://login.salesforce.com
```
  - this will pop up your browser, log in, click Allow.

Now we have your project connected to both Git and Salesforce!

## Setting up Actions!

### Create a Connected App
*This will be used to authorize deployments initiated with GitHub Actions*

In salesforce:
- Go to Setup 
- In quickfind box type in App Manager
- Click App Manager
- Click New Connected App

Connected App Name: "GitHub Actions"

API Name: "GitHub_Actions" (will be auto populated)

Contact Email: your email

Check Enable OAuth settings
- Callback URL: http://localhost:1717/OauthRedirect
- Selected OAuth Scopes
  - Manage user data via APIs (api)
  - Manage user data via Web browsers (web)
  - Perform requests at any time (refresh_token, offline_access)
Click Save

### Create a certificate and server.key

In your repository root folder create a new folder and name it: assets

Open up your bash terminal within VSC

Make sure your terminal is in the root folder of your repository

Run these commands:
- openssl genrsa -des3 -passout pass:SomePassword -out assets/server.pass.key 2048
  - Save this password somewhere, we will need it later!
- openssl rsa -passin pass:SomePassword -in assets/server.pass.key -out assets/server.key
- openssl enc -aes-256-cbc -md sha256 -salt -e -in assets/server.key -out assets/server.key.enc -k SomePassword -pbkdf2
- openssl req -new -key assets/server.key -out assets/server.csr
It will prompt you to fill in some information, Type in whatever you like, Challenge password and Optional Company name can be skipped.
- openssl x509 -req -sha256 -days 365 -in assets/server.csr -signkey assets/server.key -out assets/server.crt
Now you have a bunch of files, but you will need only two:
- server.crt
- server.key.enc

**Delete the rest.**

### Add a digital certificate to your Connected App

- Open up your salesforce setup
- Go to App Manager
- Click arrow pointing down next to GitHub Actions Connected app
- Click Edit
- Check "Use digital signatures"
- Click Select...
- Navigate to your salesforce-gha/assets folder
- Select server.crt
- Click Open
- Click Save

Restrict access through connected app to your org
- In App Manager
- Click arrow pointing down next to GitHub Actions Connected app
- Click Manage
- Click Edit Policies
- Change Permitted Users to "Admin approved users are pre-authorized"
- Click Save
- Scroll down a little
- In Profiles section, click **Manage Profiles**
- Select "System Administrator"
- Click Save

**DELETE server.crt file from assets/ folder you will not need it anymore**

Assets folder should only contain server.key.enc file

### Create a GitHub Actions workflow! Finally!

In Visual Studio Code, Bash Terminal

use these commands:
```
git add .
git commit -m "Commiting my project"
git push origin main
```
GitHub may prompt an authorization, authorize.

Now you project should be in your remote repository.
- Open up your browser
- Navigate to your repository
- At the top menu, click "Actions" tab.
- Click "set up workflow yourself"
- Copy contents of .github/workflows/main.yml **IN THIS REPOSITORY**
- Paste it in the window where you are setting up actions for **YOUR REPOSITORY**
- Click "Start Commit"
- Click "Commit new file"

### Set Up secrets for your GitHub Actions

In Salesforce go to Setup
- In Quickfind box type App Manager
- Click App Manager
- Click arrow pointing down next to GitHub Actions Connected app
- Click View
- Under "Consumer Key" click copy
- Open up your browser
- Navigate to your repository
- At the top menu, click "Settings" tab.
- Click "Secrets"
- Click "New repository secret"
  - Name: CONSUMER_KEY
  - Value: Paste your consumer key from Connected App
- Click "New repository secret"
  - Name: SFDC_USERNAME
  - Value: *your salesforce username*
- Click "New repository secret"
  - Name: ENCRYPTION_PASS
  - Value: The password you made when creating server.key (SomePassword)

## Test it out

In Visual Studio Code

Create a new Apex Class

In VSC Bash terminal run this command:
```
sfdx force:apex:class:create -n DummyClass -d force-app/main/default/classes
```
You will notice two new files in your classes folder

In your VSC Bash Terminal:
```
git add force-app/main/default/classes/
git commit -m "My first Class deployment using GitHub Actions!"
git push origin main
```

In your Browser (Git repository)
- Top menu click Actions
- You will see "My first Class deployment using GitHub Actions!"
  - click on it
- Click Depyment-On-Merge
- This is the output, result of your main.yml script

In your Salesforce org

- Go to Setup
- In quickfind box type Apex Classes
- Click Apex Classes
- You will see a newly deployd DummyClass!