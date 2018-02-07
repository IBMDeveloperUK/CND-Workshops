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

Note: the address above is for the UK region of IBM Cloud. To work with another geographic region simply swap the 'eu-gb' part for:
- 'ng' (US South)
- 'us-east' (US East)
- 'eu-de' (Germany)
- 'au-syd' (Australia)

2. Log in to the region:
```
cf login
```
3. Enter your email and password when prompted
*Note: you may be prompted to select and organisation and a space (more on these later). If so ask an instructor for help or select a non-production space to work in.*

**Deploying the application**

Now that we're set up all we have to do is type:
```
cf push
```
This command sends the contents of your working directory to the cloud controller which in turn automates the build and deployment of your application.
You should receive a stream of logs in your terminal while Cloud Foundry deploys your application.
When your application has successfully started you will be able to see the URL for your app in the line that looks something like:
```
urls: event-registration-nonanalyzed-cultivator.eu-gb.mybluemix.net
```
If you copy and paste that url in to a web browser you should be able to see your running application. It should return a webpage with the message `unable to connect to database`.

## Working with services

**Creating a service**

So why are we getting that pesky error message about not being able to connect to the database? Well that's because this application expects to be provided with database credentials through an environment variable. We could do this manually but Cloud Foundry can also manage this for us too!
To view a list of available services in your cloud platform you can use the command:
```
cf marketplace
```
*Note: this may take a minute or two to return for large lists of services*

This list shows all of the services that can be provisioned and bound (added) to the application you deployed earlier. IBM Cloud has lots of IBM and 3rd party services available in their marketplace (things like machine learning APIs, messaging and databases). This catalog may make for easier reading in the UI [here](https://console.bluemix.net/catalog/).

Let's go ahead and create a database for our application:
```
cf create-service cloudantNoSQLDB Lite cloudant-event-db
```
This command tells Cloud Foundry to create a new instance of the `cloudantNoSQLDB` database, use the `Lite` plan (which is free) and name the service `cloudant-event-db`. Cloudant is a cloud native implementation of [Apache CouchDB](http://couchdb.apache.org/). You can read more about it [here](https://www.ibm.com/uk-en/marketplace/database-management).

*Note: if you want details on the usage of a cf command you can type "cf COMMAND_NAME --help"*

**Binding a service**

We now need to tell our application about the database we just created. Cloud Foundry handles this through a concept called service binding.

To bind a service to an app we use the `cf bind-service` command:
```
cf bind-service event-registration cloudant-event-db
```
Where `event-registration` is our application name and `cloudant-event-db` is the name of our database service.

Our final step is to restart the app so that it detects the new database we've bound:
```
cf restart event-registration
```
Once your app has restarted, go to the URL of your app in the browser again to check that it's working. You should see some dummy events listed. If you forget the URL or name of an app you can use `cf apps`.

## Deploying changes

Now that we have a working application. What happens if we want to make changes? In later sessions we'll learn how to set up continuous delivery pipelines with automated testing but for now let's do it the manual way:

**Modfying the application**

1. Open the `public/index.html` file in a text/code editor of your choice.
2. Make a change to the code at line 89 where it says
```
<p>Made by the IBM Cloud team</p>
```
Replace the text inside the `<p>` tags to say that you made this application instead.
3. Save the file.

**Pushing the change**

We could push the app again by using the `cf push` command but that would overwrite our previous app. What if we wanted to run both versions instead? Application names must be unique within each org and space (think of these kind of like a working directory) so to avoid overwriting the old app we can push our changes with a new name:
```
cf push event-registration-new
```
Once this has finished deploying you will be presented with a new URL and will be able to see your changes there...
**but wait...**
...it is now saying `unable to connect to database` again.

One solution would be to create a new database and bind it to our new app. Another would be to bind our existing database to the new app. Better still though would be to fix the configuration so that all new deployments of this application connect to the database we created. We do this through the **manifest file**.

**Updating the manifest**

In your application directory you should have a file named `manifest.yml`:
1. Open the manifest file in a text/code editor.
2. Change the name of the application to `event-registration-new`:
```
name: event-registration-new
```
3. Add the following line at the bottom of the `manifest.yml` file:
```
services:
    - cloudant-event-db
```
This will let the Cloud Foundry platform know that you want it to automatically bind to an existing database called `cloudant-event-db`.
4. Save your changes to the manifest file.

You can now redeploy the new version of the application using our favourite command:
```
cf push
```

*Note: if you look closely at the logs while the app is deploying you'll notice a line like this* `Binding service cloudant-event-db to app event-registration-new in org...`

### Success

You have now successfully deployed two versions of an application - both accessible from different routes. Your bonus task is to see if you can figure out how to route **all** traffic from the original `event-registration` app to the new `event-registration-new` app. *Hint: you will need to use the "cf map-route" command.*

You can find the full specification for application manifest files [here.](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)




