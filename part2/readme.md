# Workshop 2 - Working with DevOps Services

In this workshop you will set up and configure a pipeline to get your code in to production. Any future changes you make will flow through this pipeline, allowing you to make quick but reliable changes to your production application.

## Before you begin

To complete this workshop you will need:
- To download and install [Node.js](https://nodejs.org/en/) on your local machine.
- To have an account with a Cloud Foundry provider.
- To have a [GitHub](https://github.com) account and install git.

**Installing Node.js**

1. Go to the [Node.js website](https://nodejs.org/en/) and download the installer for your OS (LTS version is recommended).
2. Open the installer and complete the instructions.
3. Check your installation works by opening up a terminal and typing `node -v`. It should print your version of node e.g. `v8.9.4`.

**Registering with a Cloud Foundry provider**

There are many [certified Cloud Foundry providers](https://www.cloudfoundry.org/certified-platforms/). For these labs, however, it will be easiest if you have an IBM Cloud account:
1. [Sign up for an account here](https://ibm.biz/BdZEbj).
2. Verify your account via the email it sends you.
3. Log in to your IBM Cloud account.

**Getting a Github account and installing git**

GitHub is a source code management tool using the open source [Git](https://git-scm.com/) version control system. There are plenty of choices but GitHub is free (for public repos) and is very popular so it makes a good choice for our workshop.

1. Go to the [GitHub](https://github.com/) site and sign up.
2. Download the appropriate version of git for your OS [from here](https://git-scm.com/downloads).
3. Install git.
4. Check your git installation worked by opening a terminal and typing `git`. You should see a list of available git commands.

## Setting up your repository

In the last workshop we just developed an application locally then threw it straight at the cloud. All good developers know this is a terrible way of working:
- what happens if my code has bugs?
- what if my build or deploy fails?
- will I overwrite other people's changes?

Version control exists for a reason and is a "best practice" for all types of software development, not just Cloud Native. (You probably already knew this!)

We're going to use our code repository as the *start* of our deployment pipeline.

1. Go to GitHub.
2. Click "New repository".
3. Give your repository a name, make sure it is public and then click "create".
4. Copy the git url of your repo. It should look something like this: `https://github.com/username/reponame.git`.

Now we need to set up a folder locally that we can write our app in:
1. `git clone YOUR_REPO_URL`
2. `cd YOUR_REPO_NAME`

Any changes to this folder will be tracked by git and can be committed to the central repository as and when we want.

## Creating the application

The purpose of this workshop is to focus on the processes and tools used to deploy your app. The app itself is pretty irrelevant so we'll create a very simple one:

1. Make sure you are in your application directory and type `npm init`. This will initiate a new Node.js repository for us to work in.
2. Accept the default values to complete the initiation.

*Note: Don't worry if you're not familiar with Node.js. The workshop will guide you through all of the code and you can always copy and paste it if you make mistakes.*

The application we are building is a simple API that takes a string and converts it to capital letters. In the real world we would never use an API for this functionality as most languages have it built-in but it works as an example for now.

To serve our API we will need a webserver. [Express](https://expressjs.com/) is a lightweight webserver for Node.js. We also need a middleware module to parse the body of an incoming POST request.

**Install Express**

Node.js has a package manager called [npm](https://www.npmjs.com/) (others are also available) which makes it super easy to install modules for our application. If we remember our 12-factor app rules, one of them is to [explicitly declare and isolate dependencies](https://12factor.net/dependencies).

- Install the express module and body-parser modules using the command `npm install --save express body-parser`.

*Note: the "--save" flag tells npm to declare this dependency. This is stored in our* `package.json` *file*.

**Writing the server code**

1. Create a new file in your application directory called `server.js`.
2. Enter the following code for your server:
```javascript
var express = require('express');
var bodyParser = require('body-parser');
var port = process.env.PORT || 3000;

var app = express();
app.use(bodyParser.text());

function convertToCaps(str) {
    return str.toUpperCase();
}

app.get('/', function(req,res) {
    res.end('Hello Cloud Native Developers');
});

app.post('/api/capitalise', function(req, res) {
    res.send(convertToCaps(req.body));
})

if (require.main === module) {
    app.listen(port, function() {
        console.log('Server listening on port: ' + port);
    });
}
```
3. Save the code in your `server.js` file.

Let's take a closer look at that code. All of the functionality of our "app" is in this function:
```javascript
function convertToCaps(str) {
    return str.toUpperCase();
}
```
The other lines of code pull in dependencies, activate middleware, specify the endpoint `/api/capitalise` and start the webserver. If you are interested in learning more about Node.js [Nodeschool](nodeschool.io) is a great free resource.

*Note: in line 3 we are expecting a port to be assigned to us over environment variables. If it doesn't detect one it'll default to 3000 (for local development). Another of the 12 factors at work (port binding).*

**Running the app locally**

Let's test this app by running it on our local machine:
1. From your terminal run the command `node server.js` from your application directory. You should see the response `Server listening on port: 3000`.
2. Open a web browser and type in `http://localhost:3000`. It should return the message `Hello Cloud Native Developers`.

**Optional**
3. If you have a REST client installed, such as [Postman](https://www.getpostman.com/), you can try posting some text to the `/api/capitalise` endpoint to test that it comes back in capitals.

## Committing your changes

Now that we have a basic application we can commit these changes to our source code repository. From your application directory run the following commands in your terminal:
1. `git add -A`
2. `git commit -m "First commit"`
3. `git push`

*Note: we have included the node_modules folder in this commit which is actually a bad practice. We would normally make use of a* `.gitignore` *file.*

If you go back to GitHub and view your repository you can see the files you just added.

## Creating a DevOps toolchain

Now that we have our code repository set up we need a way to deploy the code from that repository to the cloud. Previously we were able to run the `cf push` command from our machine to send code directly to the cloud but we can't do that from a repository such as GitHub.

We will now create and configure a toolchain to do this automatically for us:
1. If you aren't already logged in, [log in](https://console.bluemix.net/) to IBM Cloud.
2. Go to the [Toolchains](https://console.bluemix.net/devops/toolchains) section and click `Create a Toolchain`.
3. Under templates, click `Build your own toolchain`.
4. Change the toolchain name if you want to and click `Create`.

A toolchain is a set of tool integrations that helps automate several "DevOps" processes for your application.

**Adding tools to your toolchain**

Once your toolchain has created:
1. Click `Add a Tool`.
2. Select `GitHub`.

You will probably be presented with an `Authorize` button if this is your first time creating a Github integration.
3. Click `Authorize` and follow the instructions on GitHub.
4. Once you are authorised, select `existing` in the Repository Type dropdown.
5. Select your new repository in the Repository URL dropdown and tick `Track deployment of code changes`.
6. Click `Create`.

Now that you've connected your repository to the toolchain we need to add a tool to automate deployments for us:

1. Click `Add a Tool`.
2. Select `Delivery Pipeline`.
3. Give your pipeline and name and click `Create`.

## Configuring the Delivery Pipeline

You should now see a delivery pipeline tile in your toolchain under the "Deliver" heading.
- You may be presented with a warning message saying that the service is required. If so, click `Add the service` and then click `Create`.
- Close that window and return to your [toolchain](https://console.bluemix.net/devops/toolchains/) view.

**Adding the build stage**
1. Click on the `Delivery Pipline` tile and click `Add Stage`.
2. Name your stage `Build` and change to the `Jobs` tab.
3. Click `Add Job` and select type "Build".
4. The `Builder Type` dropdown will give you various build options. Leave `Simple` selected - this will use the default build type from the *buildpack* for your programming language.
5. Click `Save`.

*Note: Remember from 12-factor that we store configuration in the environment? Each stage has an environment tab where we can store config.*

**Adding the deploy stage**
1. Click `Add Stage` again, notice how this time it defaults the input settings to the previous stage (build).
2. Name your stage `Deploy` and change to the `Jobs` tab.
3. Click `Add Job` and select type "Deploy".

Cloud Foundry uses the application name as the hostname for your application. Whilst you have probably given your application a unique name within your account, there's a good chance that the name is not unique for the system domain `mybluemix.net`. Let's make sure by specifying the hostname in the deploy script. We will also specify the amount of memory our application needs.
4. Change the `Deploy Script` from:
```
#!/bin/bash
cf push "${CF_APP}"
```
to
```
#!/bin/bash
cf push "${CF_APP}" -m 128M -n UNIQUE-NAME
```
Where `UNIQUE-NAME` is a unique hostname of your choice. A good practice might be to add your initials at the front of each hostname e.g. `es-cndworkshop`. The `-m 128M` flag tells Cloud Foundry to assign 128MB of RAM to our application.
5. Click `Save`.

**Triggering the pipeline**

By default, our pipeline will run automatically every time we push new changes to our GitHub repository. We can run it manually too though:
- Click on the run button next to the build stage in your pipeline.
You should now be able to see the stages running in your delivery pipeline. The build stage should complete quickly and the deploy stage shortly afterwards.

When the deployment has succeeded you will see a `Last Execution Result` box at the bottom of your Deploy stage.
- Click on the route displayed under `Last Execution Result`.
- A new browser window will open and you should see the text `Hello Cloud Native Developers`.

Your pipeline is now all set up and ready to go. Importantly, your pipeline stores all of the outputs from your `Build Stage`. This allows us to reliably redeploy that build to multiple places or to roll back versions if we make a mistake.

*Note: This segregation of Build and Deploy stages is an important element of our 12-factor app as defined by the* [Build, Release, Run](https://12factor.net/build-release-run) *rule.*

## Changing the application

To check that our toolchain is working as intended we'll make a quick change to the code on our machine and push it up to GitHub.

1. Open the `server.js` file on your local machine.

There is a section of code that looks like this:
```javascript
app.get('/', function(req, res) {
    res.end('Hello Cloud Native Developers');
});
```
2. Change the string in the `res.end();` line to something different. E.g.
```javascript
res.end('Milk was a bad choice');
```
3. Save the file.
4. Back in your terminal run the following commands:
- `git add server.js`
- `git commit -m "changed welcome message"`
- `git push`

If you jump back to your delivery pipeline you should be able to see the stages running automatically. When the deploy succeeds your app will now display a new message when you visit your route.

## Adding tests to your application

We now have a neat way of automatically updating our cloud app just by pushing code to our GitHub repo. What happens if we accidentally push some incorrect code to our repo? It might fail in the build or deploy phase but it also might go straight to our live cloud application. Adding automated testing is an important step to check everything works as intended.

*Note: A lot of the best development teams now do completely test-driven development (TDD). It would have been better practice for us to write tests for our app BEFORE we actually wrote the app itself.*

Because our application is an API it would be easy to assume that we can only test it once it is running. We definitely need to do that but we can also test individual pieces of functionality using [unit tests](https://en.wikipedia.org/wiki/Unit_testing).

**Writing the test**

Let's write a unit test to make sure the application correctly converts text to capital letters:
1. Modify the bottom of your `server.js` file to include the following code:
```javascript
module.exports = {
    convertToCaps: convertToCaps
}
```
We're going to use a test module called [tape](https://www.npmjs.com/package/tape) to run our unit tests.
2. Install tape using the command `npm install tape --save-dev`. Since tape is only used for testing and not for production we use the `--save-dev` flag to declare it as a development dependency.
3. Create a new file in your application directory called `test.js` and add the following code:
```javascript
var server = require('./server.js');
var test = require('tape');

test('testing capitalisation', function(t) {
    t.equal(server.convertToCaps('cheeky monkey'), 'CHEEKY MONKEY')
    t.end()
})
```
4. Save your `test.js` and `server.js` files.
5. From your terminal run `node test.js`. You should get the following:
```
TAP version 13
# testing capitalisation
ok 1 should be equal

1..1
# tests 1
# pass  1

# ok
```
Which means that our unit test is passing.

**Adding the test to the delivery pipeline**

Now that we know our tests work locally we want to run them automatically in our delivery pipeline.
1. Back in your pipeline, click the gear icon on your `Build` stage and click `Configure Stage`.
2. Rename your stage `Build + Unit Test`.
3. Click `Add Job` on the jobs tab select type "Test".
4. Name your test `Unit Test` and change the test script from:
```
#!/bin/bash
#Invoke tests here
```
to:
```
#!/bin/bash
node test.js
```
5. Save the stage.

**Checking that it works**

Let's update our repo with the new code we've written to test our app. From your terminal:
1. `git add -A`
2. `git commit -m "added unit tests"`
3. `git push`

You can now watch your delivery pipeline in action as the app is built, tested and deployed.

## Deploying with zero downtime

If you were watching closely you may have noticed that your app was unavailable while you were doing each redeployment following code changes. We can avoid this by doing what's referred to as a "Blue/Green" deployment.

1. In your delivery pipeline click the gear icon on the `Deploy` stage and click `Configure Stage`.
2. Change your deployment script to the following:
```
#!/bin/bash
cf delete -f "${CF_APP}"-old
cf push "${CF_APP}"-new -m 128M -n UNIQUE-NAME-new
cf map-route "${CF_APP}"-new eu-gb.mybluemix.net --hostname UNIQUE-NAME
cf map-route "${CF_APP}" eu-gb.mybluemix.net --hostname UNIQUE-NAME-old
cf unmap-route "${CF_APP}" eu-gb.mybluemix.net --hostname UNIQUE-NAME
cf unmap-route "${CF_APP}"-new eu-gb.mybluemix.net --hostname UNIQUE-NAME-new
cf rename "${CF_APP}" "${CF_APP}"-old
cf rename "${CF_APP}"-new "${CF_APP}"
```
Where `UNIQUE-NAME` is the unique hostname you gave to your application earlier e.g. `es-cndworkshop`.

Let's take a closer look at what each of these commands is doing:
```
cf delete -f "${CF_APP}"-old
```
This will delete the old version of the app we were keeping in case of rollbacks (if it exists). Often we will want to keep this running but for the sake of this workshop we will delete it to free up space in your IBM Cloud account.
```
cf push "${CF_APP}"-new -m 128M -n UNIQUE-NAME-new
```
This is the same `cf push` command we were using earlier to deploy our application but this time we are adding `-new` to the end of the app name and hostname so that it doesn't overwrite our current app.
```
cf map-route "${CF_APP}"-new eu-gb.mybluemix.net --hostname UNIQUE-NAME
```
This command maps our live hostname to our new app. At this point traffic will be split between both the new and the current version.
```
cf map-route "${CF_APP}" eu-gb.mybluemix.net --hostname UNIQUE-NAME-old
```
This adds a new route with the tag `-old` to our current version of the app.
```
cf unmap-route "${CF_APP}" eu-gb.mybluemix.net --hostname UNIQUE-NAME
```
We then remove the live hostname from our current version so that all traffic is routed to the new version.
```
cf unmap-route "${CF_APP}"-new eu-gb.mybluemix.net --hostname UNIQUE-NAME-new
```
Next we remove the `-new` hostname from the new version of our app (as it is now our live version).
```
cf rename "${CF_APP}" "${CF_APP}"-old
cf rename "${CF_APP}"-new "${CF_APP}"
```
Finally we rename our applications to reflect the fact that our new version is live and our previous version is old.

The steps outlined above can be configured further if there are multiple instances of an app running. We can chose to add delays, ramp up or down instances and/or keep more versions of an app running.

**Checking it all works**

Try making some changes to the code again and then push them to the code repo on GitHub. This time keep refreshing the URL as your application deploys. You'll notice the app never goes down but at some point you switch over to the new version.

## Optional Extra - Additional stages

Have a go at creating extra stages for your delivery pipeline. Ideally once our app has deployed we want to run tests against it to check the API works in the live environment. After that we may want to deploy it again (if we assume the first deployment is "dev" then we can then deploy to "prod"). We may also want to deploy to other geographic regions.







