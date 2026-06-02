# SONARQUBE

## Intoduction
In this project I will use SAST tools speficifically SonarQube

SonarQube
SAST(Static Appication Security Testing) 
Analyzes your own source code without running it. Finds logic flaws, injection points, hardcoded secrets, and insecure patterns introduced by your developers.
## Creating Containers 
Install Docker Desktop App
- then open cmd and run
 ```
 wsl --install
 ```
  
## In vscode 
```sh
note this code including password are from edX labs:no actual passwords are exposed 
docker network create mynet
```
## Setting up Postgres Container
```
docker run --name postgres  -e POSTGRES_USER=root -e POSTGRES_PASSWORD=Test12345  -p 5432:5432 --network mynet -d postgres

to set up a sonarqube container on port 9000

docker run -d --name sonarqube -p 9000:9000 -e sonar.jdbc.url=jdbc:postgresql://postgres/postgres -e sonar.jdbc.username=root -e sonar.jdbc.password=Test12345 --network mynet sonarqube
```
## To set up a SonarQube Container

```
note this code including password are from edX labs:no actual passwords are exposed 
docker run -d --name sonarqube -p 9000:9000 -e sonar.jdbc.url=jdbc:postgresql://postgres/postgres -e sonar.jdbc.username=root -e sonar.jdbc.password=Test12345 --network mynet sonarqube
```


## Integrating SonarQube with Github
- In sonarqube go to projects and import from github

- configuration name: 
- Github API url: *https://api.github.com*
- Personal Access Token
- To create
1. Navigate to your GitHub account settings.
2. Click on Developer settings in the left sidebar.
3. Select Personal access tokens.
4. Click on Generate new token.
5. Provide a descriptive name for the token (e.g., "SonarQube Integration").
6. Set the expiration date as needed.
7. Select the necessary scopes for the token. For SonarQube integration, the repo scope is typically sufficient.
8. Click Generate token.
9. Copy the generated token and store it securely; you won't be able to view it again.

Setting up the app integration 
```
developer settings
new github app
hompage url: url for sonarqube
callback url: for sonarqube
deselect active for Webhook
for permissions pull request is required to be write
Then install the application to all repositiories
```
![image.png](https://github.com/khadijahW/Flash028/blob/5c3b9b76a5ae4b61f5cc08a68dad50d75f16a877/SonarSnyk/image%20(3).png)

Then in sonarqube click account 

- my account
- security and generate a new token for the repository to be scanned
- Then for that repository in github create that secret
![image.png]
Next copy the build and analysis code provided from sonarqube into a build.yml file 
- keep in mind that sonarqube is running in a docker container so the build will fail
    - download ngrok to expose the container
    - https://dashboard.ngrok.com/get-started/setup/windows
    - use the authentication token command provided after installation
    - then the command should be something like
      ```
      ngrok http <url for host>
      ```
      you will then be given a url called forwarding from the command and this will be replaced with the url in the build file
The yml file will look something like this
name: Build and Analyze
```
on:  

push:    

branches:      

- main

jobs:  

build:   

name: Build and Analyze  

  runs-on: ubuntu-latest   

 steps:    

  - uses: actions/checkout@v4        

with:         

 fetch-depth: 0  # Ensure full history for accurate analysis     

 - uses: SonarSource/sonarqube-scan-action@v4     

   env:        

  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}       

  SONAR_HOST_URL: https://<someextainfohere>.ngrok-free.app

  ```
Running the yml workflow file with Github Actions
![image.png](https://github.com/khadijahW/Flash028/blob/e3dbd2ac00feb8ff6a2645113904aa6dc16e3c6f/SonarSnyk/image.png)

The results of the SonarQube Scan
![image.png](https://github.com/khadijahW/Flash028/blob/bb26deda5b4200fd171484edb2c53e9658decadd/SonarSnyk/image%20(1).png)
