+++
author = "Harvey"
title = "Towards Lab IoT Part Two"
date = "2019-10-16"
description = "Moving towards IoT in a lab (part two - python test)"
tags = ["python", "IoT"]
+++

In order to test communication we'll set up two clients, each representing an instrument, along with a server which acts as the MQTT broker, database and visualization.  The three machines are all running ubuntu under VirtualBox.  
In this case **client1** sends *measurement1* data which resets to a random value between 20 and 50 when it receives a signal from *trigger*.  **client2** sends *measurement2* data which resets to 20 when it receives a signal from *trigger*.  The **controller** monitors *measurement1* and signals the *trigger* when *measurement1* is above 50.  The **controller** also updates influxDB with data from *measurement1* and *measurement2*.  **client1** and **client2** represent the two clients, running instruments and the server is **controller**.  Codes for each are below.

{{< figure src="/images/post-images/2019-10-16-lab_iot_02/virtual_servers.png" >}}

### controller (virtual machine 1 - ip address = 10.0.2.15)

``` python
## controller
## 
## this is the controller code that runs in the background
## it subscribes to measurement1 and when that value
## reaches the threshold of 50 it publishes a trigger
## event that client 1 and client 2 both respond to
##
## It also doubles up as the subscriber for the influxDB
## database which is code that can be run in a separate
## thread or program
## 
## publish: trigger1
## subscribe: measurement1
## response: if measurement1 > 50 then publish trigger 
 
import paho.mqtt.client as mqtt
from influxdb import InfluxDBClient
import datetime
import logging
import socket
import time
 
MQTT_BROKER = '10.0.2.15'
HOSTNAME = socket.gethostname()
IPADDR = socket.gethostbyname(HOSTNAME)
INSTRUMENT_ID = 'controller'
INFLUX_CLIENT = '10.0.2.15'
INFLUX_DATABASE = 'lab1'
 
# influxDB
influx_client = InfluxDBClient(INFLUX_CLIENT, database = INFLUX_DATABASE)
 
## define logging file
FORMAT = '%(asctime)-15s %(clientip)s %(instrumentid)-10s %(message)s'
logging.basicConfig(filename = '/home/harvey/controller.log', format = FORMAT, datefmt = '%m/%d/%Y %I:%M:%S %p', level = logging.DEBUG)
d = {'clientip': IPADDR, 'instrumentid': INSTRUMENT_ID}
 
# Define the callback to handle CONNACK from the broker, if the connection created normal, the value of rc is 0
def on_connect(client, userdata, flags, rc):
    logging.info("Connection returned with result code:" + str(rc), extra = d)
 
# Define the callback to hande publish from broker
def on_message(client, userdata, msg):
    # log the response
    logging.info("Received message, topic:" + msg.topic + "  payload:" + str(msg.payload), extra = d)
    
    # push to influxDB
    current_time = datetime.datetime.utcnow().isoformat()
    topic = msg.topic.split('/')
    measurement_type = topic[2]
    json_body = [
        {
            "measurement": measurement_type,
            "tags": {},
            "time": current_time,
            "fields": {
                "value": int(msg.payload)
            }
        }
    ]
    influx_client.write_points(json_body)
 
    # fire the trigger if measurement1 > 50
    if (msg.topic == 'sensor/expt1/measurement1'):
        if (int(msg.payload) > 50):
            client.publish("sensor/trigger/client1", payload = str(msg.payload))
 
# Callback handles disconnection, log the rc value
def on_disconnect(client, userdata, rc):
    logging.info("Disconnection returned with result code:" + str(rc), extra = d)
 
def main():
 
    # Create an instance of `Client`
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_disconnect= on_disconnect
    client.on_message = on_message
 
    # Connect to broker
    client.connect(MQTT_BROKER, 1883, 60)
 
    # Subscribe to all topics and start the loop
    client.subscribe("sensor/+/+", 0)
    client.loop_forever()
 
if __name__ == '__main__':
    logging.info('Starting Broadcast from ' + INSTRUMENT_ID, extra = d)
    main()
```

### client1 (virtual machine 2 - ip address = 10.0.2.4)

``` python
## client 1 communications
## 
## this is an MQTT client code that runs in the background
## of client 1
## publish: measurement1
## subscribe: trigger1
## response: if trigger1 then reset measurement1 to a 
## random value between 20 and 50
 
import paho.mqtt.client as mqtt
import logging
import socket
import random
import time
 
MQTT_BROKER = '10.0.2.15'
HOSTNAME = socket.gethostname()
IPADDR = socket.gethostbyname(HOSTNAME)
INSTRUMENT_ID = 'machine 1'
 
## define logging file
FORMAT = '%(asctime)-15s %(clientip)s %(instrumentid)-10s %(message)s'
logging.basicConfig(filename = '/home/harvey/client1.log', format = FORMAT, datefmt = '%m/%d/%Y %I:%M:%S %p', level = logging.DEBUG)
d = {'clientip': IPADDR, 'instrumentid': INSTRUMENT_ID}
 
# Define the callback to handle CONNACK from the broker, if the connection created normal, the value of rc is 0
def on_connect(client, userdata, flags, rc):
    logging.info("Connection returned with result code:" + str(rc), extra = d)
 
# Define the callback to hande publish from broker
def on_message(client, userdata, msg):
    global measurement
    logging.info("Received message, topic:" + msg.topic + "  payload:" + str(msg.payload), extra = d)
    if (msg.topic == 'sensor/trigger/client1'):
        measurement = random.randrange(20, 50)
        logging.info("Trigger 1: measurement reset to " + str(measurement), extra = d)
 
# Callback handles disconnection, log the rc value
def on_disconnect(client, userdata, rc):
    logging.info("Disconnection returned with result code:" + str(rc), extra = d)
 
def main():
 
    # Create an instance of `Client`
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_disconnect= on_disconnect
    client.on_message = on_message
 
    # Connect to broker
    client.connect(MQTT_BROKER, 1883, 60)
 
    # initial value of measurement
    global measurement
    measurement = 0
 
    ## start MQTT client loop
    client.loop_start()
 
    # Subscribe to a topic
    client.subscribe("sensor/trigger/client1", 0)
 
    ## continuously update measurement value every second
    ## simulates instrument operation
    while True:
        client.publish("sensor/expt1/measurement1", payload = measurement)
        time.sleep(1)
        measurement += 1
 
    # Disconnection
    time.sleep(1) # wait till all messages are processed
    client.loop_stop()
    client.disconnect()
 
if __name__ == '__main__':
    logging.info('Starting Broadcast from ' + INSTRUMENT_ID, extra = d)
    main()
```

### client2 (virtual machine 3 - ip address = 10.0.2.5)

``` python
## client 2 communications
## 
## this is an MQTT client code that runs in the background
## of client 2
## publish: measurement2
## subscribe: trigger1
## response: if trigger1 then reset measurement2 to 20 
 
import paho.mqtt.client as mqtt
import logging
import socket
import time
 
MQTT_BROKER = '10.0.2.15'
HOSTNAME = socket.gethostname()
IPADDR = socket.gethostbyname(HOSTNAME)
INSTRUMENT_ID = 'machine 2'
 
## define logging file
FORMAT = '%(asctime)-15s %(clientip)s %(instrumentid)-10s %(message)s'
logging.basicConfig(filename = '/home/harvey/client2.log', format = FORMAT, datefmt = '%m/%d/%Y %I:%M:%S %p', level = logging.DEBUG)
d = {'clientip': IPADDR, 'instrumentid': INSTRUMENT_ID}
 
# Define the callback to handle CONNACK from the broker, if the connection created normal, the value of rc is 0
def on_connect(client, userdata, flags, rc):
    logging.info("Connection returned with result code:" + str(rc), extra = d)
 
# Define the callback to hande publish from broker
def on_message(client, userdata, msg):
    global measurement
    logging.info("Received message, topic:" + msg.topic + "  payload:" + str(msg.payload), extra = d)
    if (msg.topic == 'sensor/trigger/client1'):
        measurement = 20
        logging.info("Trigger 1: measurement reset to " + str(measurement), extra = d)
 
# Callback handles disconnection, log the rc value
def on_disconnect(client, userdata, rc):
    logging.info("Disconnection returned with result code:" + str(rc), extra = d)
 
def main():
 
    # Create an instance of `Client`
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_disconnect= on_disconnect
    client.on_message = on_message
 
    # Connect to broker
    client.connect(MQTT_BROKER, 1883, 60)
 
    ## start MQTT client loop
    client.loop_start()
 
    # Subscribe to a topic
    client.subscribe("sensor/trigger/client1", 0)
 
    # initial value of measurement
    global measurement
    measurement = 0
 
    ## continuously update measurement value every second
    ## simulates instrument operation
    while True:
        client.publish("sensor/expt1/measurement2", payload = measurement)
        time.sleep(1)
        measurement += 1
 
    # Disconnection
    time.sleep(1) # wait till all messages are processed
    client.loop_stop()
    client.disconnect()
 
if __name__ == '__main__':
    logging.info('Starting Broadcast from ' + INSTRUMENT_ID, extra = d)
    main()
```

### Visualization

Running all three codes leads to continuous output which updates every second.  *measurement1* and *measurement2* are stored in the InfluxDB under database **lab1** which can be visualized using grafana (default = localhost, port 3000).  
*measurement1* and *measurement2* increment every second.  When *measurement1* reaches 50 it resets itself to a random value between 20 and 50 and resets *measurement2* to 20.

{{< figure src="/images/post-images/2019-10-16-lab_iot_02/grafana_python_out.png" >}}
