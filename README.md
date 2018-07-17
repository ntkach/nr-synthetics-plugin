# New Relic Synthetics Plugin
[![Build Status](https://travis-ci.org/kenahrens/nr-synthetics-plugin.svg?branch=master)](https://travis-ci.org/kenahrens/nr-synthetics-plugin)

This plugin runs queries against New Relic Insights to calculate things like % of locations with a failure. It works by querying Insights every 30 seconds, calculating the new metrics (like % of failing locations) and publishing the results as Plugin metrics.

## Installation and Setup
There are currently 2 options for running this plugin:
* Run the plugin directly through NodeJS
* Run the plugin as a Docker container

## Setup and Run Directly
Here are the steps to get set up with this plugin:
* Pre-requisite is NodeJS (4.x or newer) and npm
* Clone or download this repository
* Once in the directory run ```npm install```
* Setup your configuration for a single account

Once you are all set up you can run the plugin with ```npm start```

## Setup and Run as Docker Container
Thanks to @ntkach there are now some Docker files. Here's how you run through Docker.
* Pre-requisite is NodeJS (4.x or newer) and npm
* Clone or download this repository
* Once in the directory run ```npm install```
* Create the ENV variables (see the 4 required variables below)
* Build the container with something such as: ```docker build -t nr-synthetics-plugin .```
* Run the container detatched with the compose file: ```docker-compose up -d```

## Metrics Included
For every monitor that has reported data, this plugin currently will calculate and publish the following metrics:
* Component/Location/{locationName}/Success[pct]
* Component/Location/{locationName}/Failure[pct]
* Component/Location/{locationName}/Duration[ms]
* Component/Overall/Success[pct]
* Component/Overall/Success[count]
* Component/Overall/Failure[pct]
* Component/Overall/Failure[count]
* Component/Overall/Duration[ms]

## Configure for a Single Account
There is a config file default.json which defines the required keys. You have 2 options for how to configure the keys:
* Config File: Copy default.json and create your own config file
* Environment Variables: Set environment variables

### Single Account: Config file
Copy default.json and make your own file like ken-config.json. At runtime you must define an environment variable `NODE_ENV` and set to the name of your file (do not include the extension).
```
kahrens@envy5:~/dev/node/nr-synthetics-plugin$ ls -al config/
total 20
drwxrwxr-x 2 kahrens kahrens 4096 Mar 27 10:41 .
drwxrwxr-x 7 kahrens kahrens 4096 Jun 26 15:28 ..
-rw-rw-r-- 1 kahrens kahrens  359 Mar 27 10:41 ahrens.json
-rw-rw-r-- 1 kahrens kahrens  214 Mar 27 10:15 custom-environment-variables.json
-rw-rw-r-- 1 kahrens kahrens  226 Mar 27 10:15 default.json
kahrens@envy5:~/dev/node/nr-synthetics-plugin$ export NODE_ENV=ahrens
```

### Single Account: Environment Variables
If you want to query metrics from a single account and post to the same account, you can just set these 3 environment variables:
* NEWRELIC_LICENSE_KEY maps to licenseKey (for plugin to publish metrics)
* NEWRELIC_ACCOUNT_ID maps to accountId
* NEWRELIC_INSIGHTS_QUERY_KEY maps to insightsQueryKey (for plugin to query metrics)
* NEWRELIC_INSIGHTS_INSERT_KEY maps to insightsInsertKey (for plugin to publish events)

### Running the Plugin Directly
You can run the plugin directly like so. Note that it picked up my NODE_ENV that I set earlier to `ahrens`.
```
kahrens@envy5:~/dev/node/nr-synthetics-plugin$ npm start

> nr-synthetics-plugin@2.1.1 start /home/kahrens/dev/node/nr-synthetics-plugin
> node index.js

Thu, 20 Jul 2017 14:27:33 GMT - info: Synthetics Plugin version: 2.1.1 started:
Thu, 20 Jul 2017 14:27:33 GMT - info: * GUID: com.adg.synthetics.monitor.Synthetics
Thu, 20 Jul 2017 14:27:33 GMT - info: * Frequency is every 30s, cron: (*/30 * * * * *)
Thu, 20 Jul 2017 14:27:33 GMT - info: * Running as a single config.
Thu, 20 Jul 2017 14:28:00 GMT - info: Starting poll cycle with NODE_ENV Environment: ahrens
Thu, 20 Jul 2017 14:28:00 GMT - info: - Query by location: (14) locations < (55) monitors
Thu, 20 Jul 2017 14:28:01 GMT - info: - There are: 55 monitors
Thu, 20 Jul 2017 14:28:01 GMT - info: - Processing complete for : 55 monitors
```

## Running the Plugin with Forever
Or you can use forever to launch the plugin (and re-launch if it crashes):
```
kahrens:nr-synthetics-plugin kahrens$ ./node_modules/forever/bin/forever start index.js 
warn:    --minUptime not set. Defaulting to: 1000ms
warn:    --spinSleepTime not set. Your script will exit if it does not stay up for at least 1000ms
info:    Forever processing file: index.js
```

This is how you get a list of all your forever processes:
```
kahrens:nr-synthetics-plugin kahrens$ ./node_modules/forever/bin/forever list
info:    Forever processes running
data:        uid  command             script   forever pid   id logfile                          uptime      
data:    [0] UKn_ /usr/local/bin/node index.js 70782   70783    /Users/kahrens/.forever/UKn_.log 0:0:0:8.935 
```

And this is how you get the log:
```
kahrens:nr-synthetics-plugin kahrens$ ./node_modules/forever/bin/forever logs 0
data:    index.js:71019 - Mon, 22 Aug 2016 12:45:59 GMT - info: Synthetics Plugin started
data:    index.js:71019 - Mon, 22 Aug 2016 12:46:00 GMT - info: Starting poll cycle with env: multi
data:    index.js:71019 - Mon, 22 Aug 2016 12:46:00 GMT - info: getMonitorList for: Ahrens Design Group
data:    index.js:71019 - Mon, 22 Aug 2016 12:46:00 GMT - info: getMonitorList for: Demotron
```

This is how you stop your plugin, then run ```list``` to see it's no longer running.
```
kahrens:nr-synthetics-plugin kahrens$ ./node_modules/forever/bin/forever stop index.js 
info:    Forever stopped process:
    uid  command             script   forever pid   id logfile                          uptime      
[0] UKn_ /usr/local/bin/node index.js 70782   70783    /Users/kahrens/.forever/UKn_.log 0:0:8:0.212 
```

### Running the Tests
The tests work if you're using a single account and will make sure your node environment is setup properly. All you have to do to run them is:
```
kahrens:nr-synthetics-plugin kahrens$ npm test

> nr-synthetics-plugin@1.0.0 test /Users/kahrens/Documents/github/nr-synthetics-plugin
> mocha



  New Relic Insights API Test
    ✓ calls the query api (271ms)

  New Relic Plugin Make Metric Test
    ✓ makes a full metric name
    ✓ makes the complete metric JSON


  3 passing (283ms)
```
