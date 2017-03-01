# Custom IoT gateway

## Introduction
The purpose of this proof of concept is to test the viability of building our custom IoT gateway to solve the detected needs in the Edge/Bus.

This is the basic architecture we are going to test.

![Architecture](img/schema.png)

## How to
There are three independent parts to set up:
* [Set up the gateway](gateway/README.md)
* [Set up the data platform](data_platform/README.md)
* [Set up the thing](thing/README.md)


## Running the PoC
Requirements:
1. The gateway is working.
1. The MQTT broker is running.
1. The thing is flashed with the firmware.


### External MQTT broker
If you use an external MQTT broker, edit the DNAT rule in the gateway file `/etc/shorewall/rules`.

Example
```
DNAT            wifi            wired:m10.cloudmqtt.com:14411       tcp     1883
```

And restart the service.

```
service shorewall restart
```

### Listening in the data platform

http://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices

### Test

#### Reading sensors
```
# Whole device, including led
mosquitto_sub -h localhost -p 1883 -t 'BEEVA-06/SENSOR-01/#'

# Temperature
mosquitto_sub -h localhost -p 1883 -t 'BEEVA-06/SENSOR-01/temperature'

# Humidity
mosquitto_sub -h localhost -p 1883 -t 'BEEVA-06/SENSOR-01/humidity'
```

#### Using the led
```bash
# Swith the LED On
mosquitto_pub -h localhost -p 1883 -t 'BEEVA-06/SENSOR-01/LED' -m "1"

# Swith the LED Off
mosquitto_pub -h localhost -p 1883 -t 'BEEVA-06/SENSOR-01/LED' -m "0"
```
