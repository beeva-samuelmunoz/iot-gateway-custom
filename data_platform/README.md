# Data Platform Set Up

Here I show how to create a simple MQTT server to use as data platform input. It will be enough for the PoC.

## Table of contents
* [Requirements](#requirements)
* [MQTT Broker](#broker)
* [Test](#test)


## Requirements <a name="requirements"></a>
* A Computer
* Docker
* `mosquitto-clients` package (Debian/Ubuntu)


## MQTT Broker <a name="broker"></a>

* [Mosquitto](https://mosquitto.org/)

### Docker
* https://hub.docker.com/_/eclipse-mosquitto/

#### Create the container
```bash
# Install Docker image
docker pull eclipse-mosquitto

# Temporal files
data_platform
mkdir data log

# Create container
docker create -p 1883:1883 -p 9001:9001 -v config:/mosquitto/config -v data:/mosquitto/data -v log:/mosquitto/log --name iot-test-mqtt eclipse-mosquitto
```

#### Container operative
```bash
# Start the container
docker start iot-test-mqtt

# Stop the container
docker stop iot-test-mqtt

# Delete the container
docker rm iot-test-mqtt
```

### Test <a name="test"></a>
With a started container. Check it with `docker ps -a`.
Open two terminal screens.

#### Subscriber
This screen will receive the MQTT message.
```bash
mosquitto_sub -h localhost -p 1883 -t 'test-topic/#'
```

#### Publisher
This screen will send the MQTT message.
```bash
mosquitto_pub -h localhost -p 1883 -t 'test-topic' -m "This is a test"
```

If everything goes right, you should see `This is a test` in the subscriber screen.
