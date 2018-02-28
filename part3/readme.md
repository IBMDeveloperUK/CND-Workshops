# Workshop 3 - Monitoring and Logging

During this workshop you'll understand how to diagnose activities in a Cloud Foundry application. You'll look at the logs, first individually, and then from an aggregated source. You'll visualise those using [Kibana](https://www.elastic.co/products/kibana), an open-source tool, then set up availability monitoring for your app.

## Before you begin

To complete this workshop you will need:
- To download and install the Cloud Foundry CLI
- To have an account with a Cloud Foundry provider

**Installing the Cloud Foundry CLI**

1. [Go here](https://github.com/cloudfoundry/cli/releases) and grab the appropriate installer for your OS
2. Follow the installation instructions
3. Check your installation works by opening up a terminal and typing `cf help`. You should see a list of available CF commands.

**Registering with a Cloud Foundry provider**

There are many [certified Cloud Foundry providers](https://www.cloudfoundry.org/certified-platforms/). For this lab in particular, however, you will need to have an IBM Cloud account:
1. [Sign up for an account here](https://ibm.biz/BdZsn7)
2. Verify your account via the email it sends you
3. Log in to your IBM Cloud account

## Downloading the sample application

For this exercise you will use [this python application](https://github.com/IBM-Cloud/application-log-analysis). The application allows users to generate logs at various logging levels as well as setting a server threshold for catching those logs.

**Downloading using GIT**
1. `git clone https://github.com/IBM-Cloud/application-log-analysis.git`
2. `cd application-log-analysis`

**Downloading as a zip**
1. Download the [sample application here](https://github.com/IBM-Cloud/application-log-analysis/archive/master.zip)
2. Unzip the folder
3. Open a terminal (command prompt on windows) and navigate to the unzipped folder using the `cd` command e.g.
```
cd /Users/ed/Downloads/application-log-analysis
```

## Deploying the application

Ideally we'd set up a delivery pipeline like we did in the previous workshop but the purpose of this workshop is to focus on application logs and monitoring so we'll just use `cf push`.

1. `cf login`
2. `cf push APP_NAME` where APP_NAME can be any name for your app.

*Note: If you get an error it's probably because the hostname is not unique. Try changing the name of your app to something more obscure or change the hostname using the -n option.*

## Viewing Cloud Foundry logs

One of the 12 factors of building cloud applications was about [logs](https://12factor.net/logs). It's important in a cloud environment to treat these as event streams rather than discrete events, particularly as our apps should be [stateless services](https://12factor.net/processes).

While your app was pushing you would have seen a stream of logs being sent to your terminal. These logs refer to the build, staging and deployment of your app but Cloud Foundry will also collect your application logs automatically too. Anything logged to STDOUT or STDERR will be collected by the CF [Loggregator](https://github.com/cloudfoundry/loggregator).

In your terminal run the command:
```
cf logs APP_NAME --recent
```

This will return recent logs for your application which will look a lot like the logs you just saw as your application deployed.

Now run the command:
```
cf logs APP_NAME
```
Your terminal is now tailing logs directly from your application. Go to a web browser and open the url of your application (this is listed at the end of your deploy step and will usually be http://APP_NAME.eu-gb.mybluemix.net).

## Generating application logs

The application allows you to log a message at a chosen log level. The available log levels are critical, error, warn, info and debug. The application's logging infrastructure is configured to allow only log entries on or above a set level to pass. Initially, the logger level is set to warn. Thus, a message logged at info with a server setting of warn would not show up in the diagnostic output. The UI allows to change the logger setting for the server log level as well.

1. Generate several log entries by submitting messages at different levels. Change the server-side log level in-between to make it more interesting.
2. Now switch back to your terminal and you should see the new entries being streamed to the console.
<!-- Add an example of the log entry? -->
3. Finally, switch back to your application and generate some more log messages. You will need these for the next part of the workshop.

## Search and analyse the logs

Cloud Foundry does a great job of collecting logs centrally for us but a continuous stream of events is not the easiest thing for us users to understand. This is why we'd want to use a centralised logging tool so that we can search and visualise the logs. We'll use the [Log Analysis](https://console.bluemix.net/catalog/services/log-analysis) service in IBM cloud but there are plenty of other alternatives that all function very similarly.

1. Navigate to the [IBM Cloud Dashboard](https://bluemix.net).
2. Click on your application name to bring up your app dashboard.
3. On the left hand side, click on `Logs`.

You can also see the logs here in the dashboard, which you can use to filter the logs:
- Click on the filter icon in the top right and select `Application (APP)`.

This will now display only those log entries coming from your application itself. You'll notice that you can also filter by a number of Cloud Foundry components.

This view is still pretty unwieldy, however, and it would be a pain to have to search every application like this when something goes wrong.
- Click on `View in Kibana`.

This will open up a [Kibana](https://www.elastic.co/products/kibana) dashboard that is initially scoped to the application we were just on. It sits on top a central repository for all of your applications which means we can visualise all of our logs from one place.

*Note: The Log Analysis / Kibana dashboard, by default shows all available log entries from the past 15 minutes. Most recent entries are shown on the top and automatic refresh is turned off by default. The visible bar chart represents the count of messages per 30 seconds over those 15 minutes.*

**Modify the view**

The left hand side of your dashboard shows the available attributes for your logs.
1. Mouse over `message` and click the `add` button that appears next to it. This adds the actual message that was logged to the view.
2. Adjust the displayed interval by navigating to the upper right and clicking on `Last 15 minutes`. Adjust the value to `Last 1 hour`.
3. Next to the configuration of the interval is the auto-refresh setting. By default it is switched off, but you can change it.

Near the top of the dashboard is the search field. By default it will be populated with your application ID which will look something like this:
```
application_id:086d10fe-d529-4040-ae39-40429047f6e4
```
You can add operators to the search field and [define your own search queries](https://console.bluemix.net/docs/services/CloudLogAnalysis/kibana/define_search.html#define_search).
4. Add the following to the end of your search query:
```
AND message:(CRITICAL|INFO|ERROR|WARNING|DEBUG) AND message_type:ERR
```
This will filter for all of the messages we logged manually using the application's UI.
5. Click the save icon in the top right of the interface and give your search a name of `ERRlogs`.

**Creating visualisations**

You've now done some basic filtering and searching of your logs. When you have lots of logs its often easier to create visualisations so that we can easily identify areas for investigation.

1. On the top of your screen, click on `Visualize`.
2. Click on `Pie Chart` and choose `From a saved search`.
3. Choose your `ERRLogs` search you saved earlier.
4. On the left side under "Select buckets type" choose `Split Slices` and choose `Filters` under "Aggregation".
5. Change the value of "Filter 1" to `CRITICAL`.
6. Add extra filters with values of `ERROR`, `WARN`, `INFO` and `DEBUG`.
7. On the left select the "Options" tab and enable the `Donut` check box then click on the green play sign next to it.
8. In the top right corner click the save icon and save your visualisation as `DonutERR`.

Let's now create a simple metric so that we have multiple visualisations for our dashboard.

1. Click on `Visualise` at the top of the screen.
2. Pick `Metric` and choose `From a saved search`.
3. Choose your `ERRLogs` search you saved earlier.
4. Save your visualisation as `ERRCount`.

**Adding visualisations to a dashboard**

Kibana allows you to create dynamic dashboards which are a collection of your visualisations. These are interactive so as you drill down in one chart, the scope of the others will change too - pretty neat!

1. Click `Dashboard` at the top of your interface.
2. Click on the add symbol in the top right.
3. Click `DonutERR` and `ERRCount` from the list to add them to your dashboard.
4. Resize and move items around on your dashboard. You can also play around with drilling down in to them.
5. Save your dashboard in case you want to retrieve it later.

You've now created a simple dynamic dashboard based on the logs coming from one of your applications.

**Optional** - If you want to play around a little more you can try some of the following:
- Visualise non-application logs (e.g. the CF router)
- Add another application and build a combined view
- Create searches and metrics for only critical errors

## Monitoring the availability of your application

Now that you've got a robust way of working with logs you will want to monitor other aspects of your application that may not be reported in the logs. Probably the most important of these is availability i.e. is my application up and running?

IBM Cloud provides an [availability monitoring service](https://console.bluemix.net/catalog/services/availability-monitoring) that you will use in this workshop.

1. Navigate back to your application dashboard in the IBM Cloud UI.
2. On the left hand side click the `Monitoring` tab.

It's likely that at this point you don't have much availability to report because your application is very new. By default, one health check will be configured for you when you create the application.
3. Click on `View All Tests`.

You can see the test that was created automatically for you under `Synthetic Tests`.
4. Click on the default test that was provided there under `Synthetic Tests`.

This test has been configured to ping your application from London, Dallas and Melbourne so that you can get a view of how it responds to users around the world. In the top right you can see the average response time of your application and beneath that you can see the individual test instances. If you scroll further down you can see a graph of response times and any application activity.

**Adding your own test**

You will now add your own test to check availability of your app.
1. Click the back arrow in the top left to return to your monitoring dashboard.
2. Scroll down to the box titled `Synthetic Tests` and click on `Add New Test`.
3. Click on `Single Action`.

*Note: Sometimes just checking if your site is up doesn't tell the whole story. Availability monitoring lets you do advanced scripts which will carry out end to end testing on your application. This can be really useful for determining if certain functionality is working or not.*

4. Give your action a name and under the "Request" heading change the value of `URL` to the url of your application e.g. `https://my-awesome-app.eu-gb.mybluemix.net`.
5. Under the "Response Validation" heading change the first value (originally 5 seconds) to 1 second.
6. Change the second value (originally 10 seconds) to 2 seconds.

These values determine the thresholds at which our response time is at either a warning level or a critical level.
7. Click `Verify`.
8. Scroll to the bottom of the page and click on `Next`.

Now you will edit the frequency of your test to make sure it'll generate some valuable data quickly.

9. Click `Edit` next to the "Settings" heading.
10. Change the `Interval` to 1 minute and enable ALL the locations!
11. Click `Finish`.

The monitoring service will now test the url we provided for a response from these different geographic locations every minute. You will want to wait a couple of minutes for it to gather some data, now would be a good time to get another beer or ask a question!

**Checking test results**

Once your test has run a couple of times you can check the results on the monitoring dashboard.
- In the top right corner click the refresh icon to check your test has run.

If you scroll up to the top of your overview page you can see a world map with coloured circles representing the requests from each location. It's possible you'll see a yellow circle around Melbourne. If so, you can click on it to see the warning that was given.
- Scroll down your dashboard and click on the test you just created under `Synthetic Tests`.

You can view warning/critical/pass rates and each individual test instance.
- Scroll down to the response time graph and change the timescale to be a lot smaller by using the blue slider.

As your monitors gather more data this graph will be populated with response time information. It is useful because you can spot trends in how your application is performing. If all of the response times are increasing then it is likely your app is struggling under heavy load - it would be good to check the utilisation of your instances and scale up the application if appropriate.


