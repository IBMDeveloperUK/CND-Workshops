# Workshop 1 - Getting started with Cloud Foundry

In this workshop you will deploy your first application to Cloud Foundry, make changes to the application, redeploy the app and explore the Cloud Foundry CLI (command line interface).

## Before you begin

To complete this workshop you will need:
- To download and install the Cloud Foundry CLI
- To have an account with a Cloud Foundry provider

**Installing the Cloud Foundry CLI**
1. [Go here](https://github.com/cloudfoundry/cli/releases) and grab the appropriate installer for your OS
2. Follow the installation instructions
3. Check your installation works by opening up a terminal and typing `cf help`. You should see a list of available CF commands.

**Registering with a Cloud Foundry provider**
There are many [certified Cloud Foundry providers](https://www.cloudfoundry.org/certified-platforms/). For these labs, however, it will be easiest if you have an IBM Cloud account:
1. [Sign up for an account here](https://ibm.biz/BdZRBh)
2. Verify your account via the email it sends you
3. Log in to your IBM Cloud account

## Downloading the sample application

For this exercise you will use a simple event registration application. The application allows users to register for events and has an admin panel for creating events and handling registrations. More importantly, though, the application demonstrates the use of backing services for a cloud application.

1. Download the [sample application here](https://github.com/edshee/CNDWorkshops/blob/master/part1/event-registration-app.zip?raw=true)
2. Unzip the folder
3. Open a terminal (command prompt on windows) and navigate to the unzipped folder using the `cd` command e.g.
```
cd /Users/ed/Downloads/event-registration
```
4. Have a look at what's in the directory using the `ls` command (Mac and Linux) or `dir` command (Windows). It should look something like this:
```
public
index.js
package.json
manifest.yml
README.md
```

**Optional Extra - running the app locally**
If you want to try running the application locally you will need to have [Node.js](https://nodejs.org/en/) installed.
1. `npm install`
2. `npm start`
3. Visit http://localhost:8080 to view the running application

## Pushing the application to Cloud Foundry

So you now have some application code on your machine. What the code does is obviously important to you, the developer, but cloud platforms don't really care. The beauty of a platform like Cloud Foundry is that all of your applications will deploy and run in the same way regardless of what the application does or the language that it's written in.

**Logging in to Cloud Foundry**
Before we can deploy our application we need to tell the command line tool where to put it. From your terminal:
1. Set the API endpoint for the CLI:
```
cf api https://api.eu-gb.bluemix.net
```
*note: the address above is for the UK region of IBM Cloud. To work with another geographic region simply swap the 'eu-gb' part for:
- 'ng' (US South)
- 'us-east' (US East)
- 'eu-de' (Germany)
- 'au-syd' (Australia)*
2. Log in to the region:
```
cf login
```
3. Enter your email and password when prompted
*note: you may be prompted to select and organisation and a space (more on these later). If so ask an instructor for help or select a non-production space to work in.*

**Deploying the application**
Now that we're set up all we have to do is type:
```
cf push
```
You will now receive a stream of logs in your terminal while Cloud Foundry deploys your application.
When your application has successfully started you will be able to see the URL for your app in the line that looks something like:
```
urls: event-registration-nonanalyzed-cultivator.eu-gb.mybluemix.net
```
If you copy and paste that url in to a web browser you should be able to see your running application. It should return a webpage with the message `unable to connect to database`.



