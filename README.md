# `redhat-codeready-dependency-analysis`

## Overview
The The RedHat-CodeReady-Dependency-Analysis action is an interface to Red Hat CodeReady Dependency Analytics platform. 
It provides vulnerability and compliance analysis for your applications dependencies, along with recommendations to address security vulnerabilities and licensing issues.

This action reflects [Dependency Analytics VS Code plugin](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics) for Github Actions.

## Prerequisite

#### 1. Manifest File
This action scans manifest file (ex. requirements.txt, requirements-dev.txt etc.) for collecting a list of dependencies, hence prior to this action, target manifest file must be placed by user into workspace. 
The path to manifest is passed as parameters to action.

#### 2. Dependency Installation
Users need to install dependencies from manifest into a directory of workspace. 
Location of installation directory in workspace is required as parameter in the action.

#### 3. Adding Secret
To authenticate a user this action uses CRDA user key. 
This key should be saved in secrets, but before that users need to generate it.<br />
This is a one time activity to perform. 

Here are the steps to generate a CRDA user key.<br />
1. Install CRDA CLI from [here](https://github.com/fabric8-analytics/cli-tools/releases).
2. Run `crda auth` command, and it will assign user a unique id, which can be found in `~/.crda/config.yaml`. 
Users need to store the value of `crda_key` into a `Secret` to run the action.

## Action Parameters
- **manifest-file-path**: Path of target manifest file to perform analysis. `(default: requirements.txt)`
- **output-file-name**: Path of the file to save analysis report. `(default: redhat-codeready-dependency-analysis-report.json)`
- **pkg-installation-directory**: Path of a directory in workspace, where dependencies are installed.. `(default: site-package)`

## Sample Output

Result of the action is saved as JSON format in the workspace directory. 
```
{
  "report": {
    "total_scanned_dependencies": 20,
    "total_scanned_transitives": 58,
    "total_vulnerabilities": 15,
    "direct_vulnerable_dependencies": 8,
    "publicly_available_vulnerabilities": 10,
    "vulnerabilities_unique_to_snyk": 5,
    "critical_vulnerabilities": 3,
    "high_vulnerabilities": 3,
    "medium_vulnerabilities": 4,
    "low_vulnerabilities": 5,
    "report_link": "https://recommender.api.openshift.io/api/v2/stack-report/bc52c6b406f646bc84ed24fb24f5bdc9"
  },
  "exit_code": 2
}

```
This JSON data is having details about vulnerabilities and a link to view a detailed report is also provided. This JSON can be used by next action for making a decision to Fail/Pass the build.  

In the logs, a simplified report is shown, below is a sample report given in log.

```
=============================================
RedHat CodeReady Dependency Analysis Report
=============================================
Total Scanned Dependencies            :  6 
Total Scanned Transitive Dependencies :  38 
Total Vulnerabilities                 :  6 
Direct Vulnerable Dependencies        :  3 
Publicly Available Vulnerabilities    :  6 
Vulnerabilities Unique to Snyk        :  0 
Critical Vulnerabilities              :  1 
High Vulnerabilities                  :  5 
Medium Vulnerabilities                :  0 
Low Vulnerabilities                   :  0 
=============================================

Open this link to see detailed report:
https://recommender.api.openshift.io/api/v2/stack-report/bc52c6b406f646bc84ed24fb24f5bdc9 

Report is saved into file: <workspace>/redhat-codeready-dependency-analysis-report.json
Task is completed.
```

The link to detailed report will take users to a browser window having similar format as [Dependency Analytics VS Code plugin](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics). <br /> To view premium vulnerability data, users can register with Snyk token.

![Alt Text](https://raw.githubusercontent.com/fabric8-analytics/fabric8-analytics-vscode-extension/master/images/0.3.0/reg-stack-analysis.gif)

## Install the Action
```
fabric8-analytics/gh-actions@main
```

## Usage

This action expects a secret named `crda` to exist with a valid CRDA user key in `crda-key`, an attached workspace having target manifest file and its dependencies installed in a directory.

Here's an example of using one of the Actions, in this case to test a pythin-3.6 project:

```yaml
name: Workflow using RedHat-CodeReady-Dependency-Analysis action
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: setup
        run: |
          mkdir -p site-dir
          pip3 install --target=site-dir -r requirements.txt      
      - name: redhat-codeready-dependency-analysis
        uses: fabric8-analytics/gh-actions@main
        with:
          manifest-file-path: requirements.txt
          output-file-path: redhat-codeready-dependency-analysis-report.json
          pkg-installation-directory-path: site-dir
        env:
          CRDA_KEY: ${{ secrets.CRDA_KEY }}
      - name: post-action-setup
        run: |
          printf "Report file data:\n"
          cat redhat-codeready-dependency-analysis-report.json
```
