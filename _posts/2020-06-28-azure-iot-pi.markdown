---
layout: post
title: "Measuring air quality using Pi and Azure"
date: 2020-06-29 13:00:00 -0000
categories: azure iot
---
<h1>Measuring air quality using Pi and Azure</h1> 
This blog post combines two interests  of mine, the environment and developing. It was about time I used my Raspberry pi for something and just before COVID-19 shut down most of the UK in a nationwide lockdown, I was able to get my hands on a [sensor](
https://www.ebay.co.uk/itm/PM-Sensor-SDS011-Pm2-5-Laser-Dust-Sensor-PM10-Air-Quality-Module-Detection/303546342052?hash=item46acc59aa4:g:s04AAOSwwzJenp0L). The SDS011 was picked because its cheap enough and accurate enough for this project, others are available such as hackAIR, but they're more expensive.

<h3>What to measure</h3>
The sensor is going to measure the levels of two particles within the air, PM10 and PM2.5. PM is also called Particulate Matter, which is a mixture of solid particles and liquid droplets present in the atmosphere. PM2.5 refers to the atmospheric particulate matter that has a diameter of less than 2.5 micrometres, which is about 3% of the diameter of human hair, PM10 is 10 micrometers, so are both defined as fine particles.

<h3>Why measure</h3>
Having high concentrations of PM2.5 and PM10 tends to indicate high concentration of pollution in the air. A complex mixture of soot, smoke, dust and so on. Continual high levels within the air can lead to problems with breathing, irregular heart beats and other long term respiratory issues. The World Health Organisation air quality guideline stipulates that PM2.5 not exceed 10 µg/m3 annual mean, or 25 µg/m3 24-hour mean; and that PM10 not exceed 20 µg/m3 annual mean, or 50 µg/m3 24-hour mean. 

<h3>How we will measure</h3>
For this we will use a Pi 3, with Raspbian installed. The sensor to measure the particles in the air and python to run our script which will be discussed later on. For data aggregation I'm sending all the readings to Azure via an IoT Hub using the Azure python SDK, [Adafruit_IO](http://io.adafruit.com/) offers a free dashboard which is also suitable for this project, as Azure can become expensive if left running. Once the data is in IoT hub we can gather insights into the data such as prolonged exposure or high level alerts.

<h3>Implementation</h3>
Once the Pi and sensor are connected we need to read the data coming in for the correct serial port of the Pi, and then extract the bytes being sent by the sensor. We’re interested in bytes 2 and 3 for PM2.5 and 4 and 5 for PM10. We convert these from bytes to integer numbers with this line 
```python
pmtwofive = int.from_bytes(b’’.join(data[2:4]), byteorder=’little’) / 10
```
The rest of the script is explained in the comments.  In summary, collect the PM values and send the message to IoT every 20 seconds. IoT can handle vastly larger loads of data than this, but the data doesn't vary much where my sensor was located so didn't seem necessary to bloat the data feed.
To run the program, save the file on your Pi and execute
```powershell 
python3 airq.py  
```
```python
{% raw %}
import random
import time, serial
import os
 
from azure.iot.device import IoTHubDeviceClient, Message

CONNECTION_STRING = "AZURE_CONNECTION_STRING" #IOT connection string

# Define the JSON message to send to IoT Hub.
 
MSG_TXT = '{{"PM25": {pmtwofive},"PM10": {pmten}}}'
ser = serial.Serial('/dev/ttyUSB0') #connect to the  PM Sensor

def iothub_client_init():
    # Create an IoT Hub client
    client = IoTHubDeviceClient.create_from_connection_string(CONNECTION_STRING)
    return client

def iothub_client_telemetry_sample_run():

    try:
        client = iothub_client_init()
        print ( "IoT Hub device sending periodic messages, press Ctrl-C to exit" )

        while True:

            data = []
            for index in range(0,10):
             datum = ser.read()
             data.append(datum)
    
            pmtwofive = int.from_bytes(b''.join(data[2:4]), byteorder='little') / 10
            pmten = int.from_bytes(b''.join(data[4:6]), byteorder='little') / 10
            
            # Build the message with telemetry values. 
            msg_txt_formatted = MSG_TXT.format(pmtwofive=pmtwofive,pmten=pmten)
            message = Message(msg_txt_formatted)
            
            # Add a custom application property to the message.
            # An IoT hub can filter on these properties without access to the message body.
            if pmtwofive > 10:
              message.custom_properties["twofiveAlert"] = "true"
            else:
              message.custom_properties["twofiveAlert"] = "false"

            # Send the message.
            print( "Sending message: {}".format(message) )
            client.send_message(message)
            print ( "Message successfully sent" )
            time.sleep(20)

    except KeyboardInterrupt:
        print ( "IoTHubClient sample stopped" )

if __name__ == '__main__':
    print ( "IoT Hub Quickstart #1 - Simulated device" )
    print ( "Press Ctrl-C to exit" )
    iothub_client_telemetry_sample_run()
{% endraw %}
```
If you don't have a pi I would recommend running [DustViewerSharp](https://github.com/Galch/DustViewerSharp) locally with the sensor connected to your pc. You can view all the same data.