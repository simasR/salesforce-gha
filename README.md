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
#### Create a dev org!
Lets create a salesforce [developer org](https://developer.salesforce.com/signup) first.

#### Set up a Git repository.
If you dont already have a GitHub account Sign up [here](https://github.com/).
- Create a new repository name it however you like, for example: salesforce-gha.
- Select "Add a README file" checkbox
- Click Create Repository

#### Clone git repository.
On your pc find a place where you want your project to be stored.
- Right click your mouse
- Select: Git Bash Here.
  - This will open up a Bash Terminal
- Open your browser window where your git repository is
  - click "Code"
  - next to a URL there is copy button, click it.
- Go back to the Bash Terminal you have just opened.
  - type in: git clone <paste that url>  (Don't include < and >)
Now you cloned your Empty repository to your machine.

#### Create an sfdx project and Set up your Visual Studio Code (VSC)
- Create a folder for your project to be placed in. 
- Within your project folder create a file called package.json containing:
```
{
    "name": "yourProjectName"
}
```
- Set up workspace: 
  - File -> Add folder to workspace...
  - select folder you have created for the project

- Set up terminal:
  - Terminal -> New Terminal
  - If Bash terminal was not selected by default I suggest clicking arrow pointing down next to "+" at the top right of the terminal and selecting Git Bash

- Create an sfdx project
  - In terminal run this command:
```
sfdx force:project:create --projectname GhaSalesforce --manifest
```

This will create a folder with a bunch of files. 
Now we want to merge Git repository folder with Project folder.
Move contents of your **Project folder** into Repository folder.

- Authorize your org
  - In terminal run this command:
```
sfdx auth:web:login --setalias myDevOrg --instanceurl https://login.salesforce.com
```
  - this will pop up your browser, log in, click Allow.

Now we have Your project connected to both Git and Salesforce!