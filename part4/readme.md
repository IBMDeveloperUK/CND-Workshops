# Workshop 4 - Securing dependencies and tracing microservices

In this workshop you will trace communications between microservices, add security checks to the code that you deploy and then secure your cloud endpoints. By the end of the workshop you should have an understanding of some of the ways to manage complexity in a cloud native deployment.

## Before you begin

To complete this workshop you will need:
- To have an IBM Cloud account.
- To have a [GitHub](https://github.com) account and install git.
- To have a [Snyk](https://snyk.io) account.

**Registering with a Cloud Foundry provider**

There are many [certified Cloud Foundry providers](https://www.cloudfoundry.org/certified-platforms/). For these labs, however, you will need to have an IBM Cloud account:
1. [Sign up for an account here](https://ibm.biz/BdZfyY).
2. Verify your account via the email it sends you.
3. Log in to your IBM Cloud account.

**Getting a Github account and installing git**

GitHub is a source code management tool using the open source [Git](https://git-scm.com/) version control system. There are plenty of choices but GitHub is free (for public repos) and is very popular so it makes a good choice for our workshop.

1. Go to the [GitHub](https://github.com/) site and sign up.
2. Download the appropriate version of git for your OS [from here](https://git-scm.com/downloads).
3. Install git.
4. Check your git installation worked by opening a terminal and typing `git`. You should see a list of available git commands.

**Registering with Snyk**

Snyk is a dependency security tool. It helps you use open source code in your projects without worrying about the vulnerabilities you may be introducing.

1. Go to the [Snyk](https://snyk.io) site and hit sign up.
2. Sign up with your GitHub account.
3. Click `Continue Sign Up`

*Note: No need to configure any integrations yet, we'll do that later on in the lab.*

## Tracing service communications

To track the communications between our cloud microservices we're going to use an open source tool called [Zipkin](https://zipkin.io/).
![](https://zipkin.io/public/img/web-screenshot.png)

Zipkin is a distributed tracing system. It helps gather timing data needed to troubleshoot latency problems in microservice architectures. It manages both the collection and lookup of this data.

We're going to set up a Zipkin server first to collect and display metrics. After that we'll deploy a simple application that provides tracing information to that server.

**Downloading the server executable**

1. Open up a terminal window and create a new folder to work from e.g. `mkdir cndpart4`.
2. Navigate in to that folder using the `cd` e.g. `cd cndpar4/`.
3. Download the zipkin executable jar in to this directory:
```
curl -sSL https://zipkin.io/quickstart.sh | bash -s
```
If the above command doesn't work you can download the file [here](https://search.maven.org/remote_content?g=io.zipkin.java&a=zipkin-server&v=LATEST&c=exec). Just make sure you save it to the directory you're in.

**Logging in to Cloud Foundry**

Try the command `cf t` in your terminal - if it responds with your IBM Cloud account as the target then you're already logged in and can skip this section.

1. Log in to Cloud Foundry:
```
cf login
```
If you are prompted for an API endpoint input the following:
```
https://api.eu-gb.bluemix.net
```
Note: the address above is for the UK region of IBM Cloud. To work with another geographic region simply swap the 'eu-gb' part for:
- 'ng' (US South)
- 'us-east' (US East)
- 'eu-de' (Germany)
- 'au-syd' (Australia)

3. Enter your email and password when prompted
*Note: you may be prompted to select and organisation and a space (more on these later). If so ask an instructor for help or select a non-production space to work in.*

**Pushing the server app**

In your terminal make sure you are in the directory you downloaded the jar to earlier and then run the following command:
```
cf push UNIQUE_NAME -b liberty-for-java -p zipkin.jar -m 200M
```
Where `UNIQUE_NAME` should be replaced by a unique app name (e.g. es-zipkin-cnd), `-b liberty-for-java` specifies the java buildpack we want to use, `-p zipkin.jar` provides the executable path and `-m 200M` specifies 200MB of memory for the app.

Once your app has started you can visit it at the url listed in the logs (e.g. es-zipkin-cnd.eu-gb.mybluemix.net). You should see the Zipkin dashboard with some filters and a blue `Find Traces` button. Make a note of this url because you'll need it later to send traces to.

**Creating the microservices**

Now that we have a tracing server up and running let's deploy some microservices for the server to monitor. The application you're going to deploy consists of two very simple microservices. The first (front end) receives browser reqeusts and routes them to the second (backend). That backend service responds with the current date and time and the front end then serves that back to the browser. Once deployed, we will be able to trace what's going on within our system.

1. Go to sample code on GitHub, found here: https://github.com/edshee/zipkin-js-example/
2. In the top right corner click on `Fork`. This will create a copy of the repository in your own GitHub account which will be needed later.
3. Click the green `clone or download` button and copy the url provided. It should look something like `https://github.com/user/zipkin-js-example.git`
4. From your terminal window run the command:
```
git clone https://github.com/user/zipkin-js-example.git
```
making sure you use the URL you just copied.
5. Navigate in to the new directory using: `cd zipkin-js-example/`.

This code repo has two folders, one for each of the services we are going to deploy. Let's deploy the front end service first:
1. `cd frontend/`
2. Now deploy this app on Cloud Foundry using the following command:
```
cf push -n UNIQUE_NAME
```
Where `UNIQUE_NAME` is a unique hostname for the service (e.g es-zipkin-frontend).

*Note: try to remember this url, you'll need it later.*

Now let's deploy the backend service:
1. Navigate in to the backend folder using `cd ../backend/`.
2. Deploy the app in the same way you did with the front end:
```
cf push -n UNIQUE_NAME
```
Where `UNIQUE_NAME` is a unique hostname for the service (e.g es-zipkin-frontend).

*Note: we've specified the hostnames for each app but not given them a name. That was all taken care of using the manifest.yml file. If you looked closely at the deployment you may have seen that it's named your services app-frontend and app-backend.*

Both services are now up and running but they currently don't know about the Zipkin server we created earlier so they don't have anywhere to send tracing information to. Let's fix that by setting some environment variables using:
- ```
cf set-env app-frontend ZIPKIN_URL 'YOUR_ZIPKIN_SERVER.eu-gb.mybluemix.net'
```
and
- ```
cf set-env app-backend ZIPKIN_URL 'YOUR_ZIPKIN_SERVER.eu-gb.mybluemix.net'
```
Don't forget to replace `YOUR_ZIPKIN_SERVER` with the url from the first step of the workshop.

Finally we need to tell the front end about the back end service:
- ```
cf set-env app-frontend BACKEND_URL 'YOUR_BACKEND_URL.eu-gb.mybluemix.net'
```

Now all we need to do is restart the applications so that they can connect to each other:
- Use `cf restart app-frontend` and then `cf restart app-backend`.

*Note: remember from our 12-factor app design that we should be storing all config in the environment. Setting these environment variables is a prime example of that - it means we could move this application to another environment, have it report tracing to a different server and not make any code changes at all.*

**Viewing the tracing information**

Your services are now all set and should be connected to your Zipkin server. To view some results we'll need to generate them first.

1. Go to your front end url (e.g. es-zipkin-frontend.eu-gb.mybluemix.net). You should see some text with today's date and time.
2. Refresh a few times to generate multiple requests.
3. Now go to your Zipkin server url (e.g. es-zipkin-cnd.eu-gb.mybluemix.net) and you should be presented by the dashboard.

*Note: if you can't remember your app urls you can always find them by using the* `cf apps` *command.*
4. Click on the big blue `Find Traces` button.

You'll see a list of the requests that you made from your browser. You can see how long each request took from start to finish and which services were involved in handling the request.
5. Click on one of the traces - you'll see a network diagram showing the communication between each service and how long each call took. This can be really important in debugging slow performance as it helps you identify which services are taking the longest to respond and how long the calls take between each service.
6. Click on `Dependencies` at the top of the screen.

Zipkin also automatically detects which services depend on each other. For our example the visualisation looks incredibly simple but we can see that the `frontend` service depends on the `backend` service. If we had a much more complicated deployment with many microservices it can often be really difficult to figure out dependencies - that's where using something like Zipkin is great!

## Adding security checks to your code

Nearly all languages now come with some form of dependency management tool (npm/yarn for node.js, pip for python, rubygems for ruby etc...). These make it super easy to pull functionality in to your application and can save hundreds of development hours. The downside of this is that developers rarely know much at all about the code they're including in their application. Every time we introduce a new library we increase the chance of including an unknown vulnerability.

We're going to use a tool called [Snyk](https://snyk.io) to check our code is safe to deploy.

Let's make sure the application you just deployed is actually safe:
1. Go to https://snyk.io/org/ and click on the `Connect to GitHub` integration.
2. Select `public repos only` and click authorize.
3. Snyk now asks you what repositories you want to test. Select the `zipkin-js-example` repository that you created earlier (by forking).
4. Click `Add selected repositories to snyk`.

Snyk will now automatically scan the repository to see what dependencies you have included. It'll then look at what dependencies those dependencies have, iterate over that tree and then check every single part of the dependency tree for know vulnerabilities. You'll notice that this report comes back with one high severity vulnerability in our backend service.

5. Click on where it says `backend/package.json`.

You'll see lots of useful information about the vulnerability. You can see that this one has been introduced by using a module called `qs` which is a common query string parser. We can see that the way to fix this vulnerability would be to upgrade `qs` from version 4.0.0 to 6.0.4 but what's really cool about Snyk is that it can do this for you.

6. Click on `Open a fix PR`.

You'll notice that it automatically redirects you to GitHub where Snyk has opened a pull request on your code repository.

7. Click on `Merge pull request` and `confirm merge`.

Success! We're now protected from that pesky vulnerability we didn't even know we had. Any new commits to this repo will automatically be scanned and you'll be notified if anything you do introduces new vulnerabilities. You can also do some cool things like [scan live running applications on Cloud Foundry](https://snyk.io/docs/cloud-foundry).
