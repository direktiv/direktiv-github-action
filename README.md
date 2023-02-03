# Direktiv Synchronisation Action for GitHub

The Direktiv GitHub Action syncrhonises the GitHub repository on which the action is executed with the Direktiv instance and namespace (or folder).

## Prerequisites

Make sure that you have

* GitHub account with Actions enabled
* Direktiv Authenticaiton Token
* Direktiv URL
* Direktiv namespace (and optionally the full path to the directory if syncing to a directory)

## Create a Direktiv Authentication Token

Since we're configuring GitHub to run an action on Direktiv (i.e. synchronise the repository), authentication is needed to the Direktiv instance. If the Open Source version has been deployed, then the authentication token is the "apikey" value configured during the installation (it's also used to authenticate when logging into the frontend).

The create an authentication token for the Enterprise Edition, the following steps can be followed:

1. Make sure you have logged into Direktiv and have access to the namespace you want to synchronise. 
2. Click on the "Permissions" option in the left-hand menu. This will load the Group Policy and Policy Editor options. 
3. In the top-right corner of the "Group Policy" window, click on the "+Auth Token" option. 
4. The following options are needed to configure the Authentication Token:
    * **Lifetime**: the duration for which this token should be valid in ISO861 format. Example, a token that needs to last for 1 year would be "P1Y". To represent a duration of "three years, six months, four days, twelve hours, thirty minutes, and five seconds": "P3Y6M4DT12H30M5S"
    * **Token description**: describe the use for the token, as it will be displayed as a pop-up when hovering over the token once created
    * **Permissions**: select the permissions to allocate to the token. For Git integration the minimal permissions should `explorerManage` & `explorerView`. 

> NOTE: The token is generated and copied to the clipboard. Please take note of the token as it is not recoverable once created.

<img src="images/permissions.png?raw=true" width="40%" height="auto" align="middle">
   
## How the Direktiv action works
The GitHub action is executed on a push event and therefore starts whenever a Git commit happens. The Workflow has a single action that executes a HTTP(S) POST to the Direktiv instance with the following structure:

```bash
curl -X POST "${{ env.DIREKTIV_URL }}/api/namespaces/${{ env.DIREKTIV_NAMESPACE }}/tree${{ env.DIREKTIV_DIRECTORY }}?op=sync-mirror&force=true" -H "direktiv-token: ${{ secrets.DIREKTIV_TOKEN }}"
```

It uses the Direktiv Authorization Token to authenticate to Direktiv and then syncrhonises via its trigger. The result is that all the details from the Git push are transferred to the Direktiv mirror.

## How to use the Direktiv GitHub action

An example of workflow

```yaml
name: Direktiv namespace synchronisation

# Controls when the workflow will run
on:
  push:
    branches: 
    - [ "main" ]

env:
  DIREKTIV_URL: ''
  DIREKTIV_NAMESPACE: ''
  DIREKTIV_DIRECTORY: ''

jobs:
  run-updater:
    runs-on: ubuntu-latest
    steps:
    - name: REST API with curl to ${{ env.DIREKTIV_URL }}
      run: |
        curl -X POST "${{ env.DIREKTIV_URL }}/api/namespaces/${{ env.DIREKTIV_NAMESPACE }}/tree${{ env.DIREKTIV_DIRECTORY }}?op=sync-mirror&force=true" -H "direktiv-token: ${{ secrets.DIREKTIV_TOKEN }}"
```

## Environmental variables
* A secret with name DIREKTIV_TOKEN and value your Direktiv API token ( https://knowledge.direktiv.io/git-authentication-with-access-tokens )
* An environment variable called DIREKTIV_URL with a value of the Direktiv instance: `https://{fqdn}:{port}`
* An environment variable called DIREKTIV_NAMESPACE with a value of the Direktiv namespace: `workflows`
* An optional environment variable called DIREKTIV_DIRECTORY with the full path for the sync: `/workflows/directory`

Click the Done button to save your changes and commit.

Now next time you commit anything in your GitHub repository the Direktiv action will also execute.

## Outputs
The action will report if the action was successful. This will be standard `curl` output
