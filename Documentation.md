# Purpose:

To optimize the deployment process from deployment 2 by fully automating the CI/CD pipeline and related tasks.

This deployment uses Jenkins to fully automate the build, test, and deployment process of a python URL shortening application. The process uses the awsebcli utility to deploy a successful build to AWS Elastic Beanstalk.

## Deployment Files:

The following files are needed to run this deployment:

- `application.py` The main python application file
- `test_app.py` Tests used to test application functionality; used in Jenkins Test phase
- `requirements.txt` Required packages for python application
- `urls.json` Test URLS for application testing
- `Jenkinsfile` Configuration file used by Jenkins to run a pipeline
- `README.md` README documentation
- `Documentation.md` Documentation on how to run the deployment
- `static/` Folder housing CSS files
- `templates/` Folder housing HTML templates
- `images/` Folder housing deployment artifacts

# Steps:

1.  Launch an EC2 Instance and install Jenkins

    - Jenkins, an open-source software automation server, will be used to automate the process of pulling in the source code repository, building and running the application, and testing the application.
    - Install Jenkins Server using instructions at this [link](https://pkg.jenkins.io/debian/)
    - Install required version of python:
      - `sudo apt install python3.10-venv`
    - Browse to http://<instance public IP>:8080 to access the Jenkins server
      - Follow on-screen instructions to complete initial login

2.  Setup Multibranch Pipeline

    - From Dashboard, select a new item > Create Name > Mulitbranch Pipeline option
      ![deploy3_1](images/)<br>
    - Under Configuration > General, set branch source to ‘GitHub’ option
    - Set Branch sources:
      - Credentials: [How to setup Github Access Token](https://docs.github.com/en/enterprise-server@3.8/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
      - Repository HTTPS URL: `<Github Repo URL>`
    - Apply and Save

3.  Run Pipeline

    - Select Build Now
    - During the pipeline run we're lookng for a successful build and test.

4.  Setup AWS CLI

    - Create a [AWS access and secret access key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) credentials.
    - Download and install the AWS CLI files.

      > `curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip'` `unzip awsliv2.zip`

      > `sudo ./aws/install`

      > `aws configure`

      - `Enter AWS Access Key ID: <aws access key>`
      - `Enter AWS Secret Access: <aws secret key>`
      - `Region Name: us-east-1`
      - `Output Format: json`

5.  Setup AWS Elastic Beanstalk CLI for Jenkins User

    - Create a password for the jenkins user
      > `sudo passwd jenkins`
    - Switch to the jenkins user account
      > `sudo su - jenkins -s /bin/bash`
    - Install the AWS ebcli
      > `pip install awsebcli --upgrade --user` > `export PATH=$PATH:$HOME/.local/bin`
    - Configure awsbscli utility
      > `aws configure`
      - `Enter AWS Access Key ID: <aws access key>`
      - `Enter AWS Secret Access: <aws secret key>`
      - `Region Name: us-east-1`
      - `Output Format: json`

6.  Deployment Setup to AWS Elastic Beanstalk from Jenkins

    - Navigate to the Jenkins multibranch pipeline directory
      > `cd workspace/Deployment3_1_main/`
    - Initialize directory for awsebcli
      > `eb init`
      - `us-east-1`
      - `Press enter for default`
      - `Environment: Python version 3.9`
      - `Code Commit: select No`
      - `SSH: select No`
    - Create AWS Elastic Beanstalk:

      > `eb create`

      - `Press enter for default`
      - `Press enter for default`
      - `Press enter for default`
      - `Spot Fleet: select No`
      - `SSH: select No`

    - Verify deployment to Elastic Beanstalk
      ![deploy3_1_eb_deploy](images/)<br>

7.  Add Deployment Phase to Jenkins Pipeline

    - Update the jenkinsfile with a new stage to represent the deployment phase. The `eb deploy` will be run during this phase to upload the application files to Elastic Beanstalk.
      ![deploy3_1](images/)<br>
    - Run Jenkins pipeline to test deployment phase.

# System Diagram:

CI/CD Pipeline Architecture [Link](https://github.com/kaedmond24/python_url_shortener_app_deployment_3/blob/main/c4_deployment_3_1.png)

# Issues:

Version 2 of the application was successfully deployed and accessible. However, during testing of the applications functionality an error was recieved. The error, internal server error, is a 500 code error that indentifies a general issue the server is experiencing.

The first step in troubleshooting this issue was to pull the logs from Elastic Beanstalk. A request for the last 100 lines of logs should be enough to get a picture of the issue. Once download, a review of the logs revealed an issue within the Python application.py file.

        ```Sep 22 20:44:55 ip-172-31-88-229 web[2665]: [2023-09-22 20:44:55,938] ERROR in app: Exception on /your-url [POST]
        ...
        Sep 22 20:44:55 ip-172-31-88-229 web[2665]:  File "/var/app/current/application.py", line 20, in your_url
        Sep 22 20:44:55 ip-172-31-88-229 web[2665]:    urls = json.loads(urls_file)
        Sep 22 20:44:55 ip-172-31-88-229 web[2665]:  File "/usr/lib64/python3.9/json/__init__.py", line 339, in loads
        Sep 22 20:44:55 ip-172-31-88-229 web[2665]:    raise TypeError(f'the JSON object must be str, bytes or bytearray, '
        Sep 22 20:44:55 ip-172-31-88-229 web[2665]: TypeError: the JSON object must be str, bytes or bytearray, not TextIOWrapper```

The issue detected was a Python TypeError referencing some type of issue with a json object on line line 20. Checking the extent of the error can give some insight into whether a quick fix is needed or some logic needs to be adjust. After reviewing the application.py file it determined that an incorrect json method was attempted to be used to parse a json object. The json.lods() method was used which parses a json string object. However, a regular json object is being passed which needs the json.load() method to parse the data corectly.

Due to the current deployment of application version 2 being broken, a application version rollback would be the best course of acction to take in the moment. With the issue to be fixed buried in the code, getting the current version into a working state could talonger that the SLA would allow. Therefore, redploying version 1 of the application would give users a working version while version 2 is modified with the propor fixes.

# Optimization:

1. This deployment is an improvement over deployments 1 and 2. The addition of webhooks and the awseb cli has optimized the deployment process by adding full automation. However, further improvements can be implemented to further refine the process. I believe having a delivery phase where multiple versions of the application can be created and stored would be beneficial. In the case of any issue with a current version in production, a previous version of the application can be quickly redeployed.
