# Bluemix Code Labs - Watson IoT

During this lab, you will use [Watson IoT][watson_iot] to collect live sensor data from smartphones & calculate the energy generated per user.

You can see a version of this app that is already running [here](https://stepiot.mybluemix.net/red/). 
Players can view their smartphone sensor data [here](https://ibm.biz/stepiot). You can deploy your own version of this application by clicking [here](https://bluemix.net/deploy?repository=https://github.com/ibm-messaging/discover-iot-sample). Just ensure that both applications are using the same Watson IoT service :)

 ![Overview](http://i.imgur.com/vE6Wpcl.jpg)

So let’s get started. The first thing to do is to build out the shell of our application in Bluemix.

## Creating a [IBM Bluemix][bluemix] Account

1. Go to https://bluemix.net/
2. Create a Bluemix account if required.
3. Log in with your IBM ID (the ID used to create your Bluemix account) 

**Note:** The confirmation email from Bluemix mail take up to 1 hour.

## Deploy this Node-RED IoT starter application on Bluemix

[![Deploy to Bluemix](https://bluemix.net/deploy/button.png)](https://bluemix.net/deploy?repository=https://github.com/naisofly/stepiot)

## Add a relational Database

1. From the application Dashboard, click on “connections” and then on “connect new.” 
Now, we will add a relational database to the application:

 ![Dashboard](http://i.imgur.com/yCTi1g3.png)
 
2. Search for “dashDB” (using the search field), and then select "dashDB for Analytics"

 ![Catalog](http://i.imgur.com/31Dhfc2.png)
 
3. Click on create:

 ![dashDB](http://i.imgur.com/cNRquS5.png)
 
4. Restage your application. Once it completes, you now have everything in place to capture and store the data

After completing the steps above, you are ready to test your application. Start a browser and enter the URL of your application.

                  <your-application-name>.mybluemix.net

You can also find your application name when you click on your application in Bluemix.

## Capture and store the data using NodeRED

1. Start your NodeRED instance by opening the URL <yourUniqueName>.mybluemix.net/red – w
<yourUniqueName> is the one you’ve chosen in the previous step. You should see a little sample app:

 ![NodeRED](http://developer.ibm.com/iotplatform/wp-content/uploads/sites/24/2016/08/Untitled9.png)
 
2. Select all the nodes on the panel and click on delete, basically removing the sample app because we will create our own flow now.
From the left hand palette, drag the IBM IoT input node (don’t use the output node) and the debug node to the pane and connect them by clicking on the little connection circles on each node. The flow should look like this:

 ![NodeRED](http://developer.ibm.com/iotplatform/wp-content/uploads/sites/24/2016/08/Untitled10.png)
 
3. Double-click on the IoT node, select “Bluemix Service” under authentication and click on “done”. 
(This basically tells the node to get the credentials to connect to the MQTT broker via CloudFoundry credentials injection, 
which is a very handy feature because in the Bluemix UI the application is connected to the MQTT broker, 
is the Watson IoT service.) This is the same as you’ve connected dashDB to the application in the previous section:

 ![NodeRED](http://developer.ibm.com/iotplatform/wp-content/uploads/sites/24/2016/08/Untitled11.png)

4. Click on deploy and select the debug pane below the deploy button. Now go to https://ibm.biz/stepiot on your smartphone. Provide an 8 character unique device name & password to start sending sensor data to the Watson IoT service.
 On NodeRED, you should see this data in the debug output:

 ![NodeRED](http://i.imgur.com/aKnAJvt.png)

Congratulations! We are almost done, now we have to only stream this data to the data base and analyze it.

## Stream IoT sensor data to a RDBMS (dashDB)

1. From the NodeRED panel select the “function” node because we have to flatten the hierarchical JSON messages to a relational scheme first. Just put it in between the two existing nodes and connect accordingly, so that the result looks like this: 

 ![NodeRED](http://developer.ibm.com/iotplatform/wp-content/uploads/sites/24/2016/08/Untitled14.png)

2. Double-click on the function node and insert the following JavaScript code: 

  ```none
  msg.payload = { 
  X : msg.payload.d.ax,
  Y : msg.payload.d.ay,
  Z : msg.payload.d.az, 
  SENSORID : msg.payload.d.id } 
  return msg;
  ```

3. Click on “done” and click “deploy” . On the debug pane, the output should now be formatted to show only accelerometer data and the sensorID.
Now we are prepared to store the data in a relational database.

## Prepare the table in dashDB

1. From the Bluemix console click on your application, then connections, and then on dashDB. Click "Open":

 ![dashDB](http://i.imgur.com/EQCBPBR.png)
 
2. Click on “Run SQL”, insert to following snippet of code, and click on RUN:
CREATE TABLE SHAKESHAKE (SENSORID VARCHAR(23),X DECIMAL, Y DECIMAL, Z DECIMAL)

 ![dashDB](http://developer.ibm.com/iotplatform/wp-content/uploads/sites/24/2016/08/Untitled19.png)

## Stream the data into dashDB

1. Open NodeRED again. From the left panel, choose the dashDB output connector, and connect it to the output of the function node: 

 ![dashDB](http://developer.ibm.com/iotplatform/wp-content/uploads/sites/24/2016/08/Untitled21.png)
 
2. Double-click on the dashDB node, and select your DB service (There should only be one unless you’ve connected more than one dashDB to this application in the Bluemix console) Enter “SHAKESHAKE” as the destination table. 
Since the NodeRED schema now matches the DB schema everything is done in order to stream data into dashDB.

 ![dashDB](http://developer.ibm.com/iotplatform/wp-content/uploads/sites/24/2016/08/Untitled22.png)
 
3. Click on "done", then on "Deploy"

 Now it’s time to check if data arrives in the table. 
 Please ensure the app on your smart phone is still sending data which can also be checked on the debug pane of NodeRED.
 
## Check for data arriving in the database

1. Return to the dashDB user interface, and click on “Run SQL”.

2. Enter the following SQL query - "SELECT * from SHAKESHAKE". You should be able to see the sensor data from your phone displayed here.

3. You can now get your friends to visit the same app of their smartphones by visting https:/ibm.biz/stepiot & ask them to shake their device in all possible directions.

4. You can check for the winner using the following SQL statement in dashDB - "SELECT SENSORID, SUM(X)+SUM(Y)+SUM(Z) as ENERGY from SHAKESHAKE group by SENSORID ORDER BY ENERGY DESC"

To start over just run DELETE FROM SHAKESHAKE and you are ready for the next game!

# Congratulations

You have completed the Watson IoT Lab! :bowtie:

 ![Congratulations](https://media.giphy.com/media/aLdiZJmmx4OVW/giphy.gif)

[sign_up]: https://bluemix.net/registration
[bluemix]: https://console.ng.bluemix.net/
[wdc_services]: http://www.ibm.com/watson/developercloud/services-catalog.html
[watson_iot]: http://discover-iot.eu-gb.mybluemix.net/#/play
[cloud_foundry]: https://github.com/cloudfoundry/cli
