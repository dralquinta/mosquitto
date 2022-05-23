

Loosely Based on: 
- https://serverfault.com/questions/1045712/how-to-setup-mosquitto-mqtt-broker-in-kubernetes 
- https://blogs.oracle.com/developers/post/install-and-run-mosquitto-on-a-kubernetes-cluster
- https://blogs.oracle.com/developers/post/installing-securing-mosquitto-for-encrypted-mqtt-messaging-in-the-oracle-cloud



to try: 

`sudo yum install mosquitto`

Expected output: 

Pub: 

```shell
mosquitto_pub -t demo/one -h 129.151.125.171 -u $MOSQUITTO_USER -P $MOSQUITTO_PASSWORD -p 80 -m '{"message": 1}'
mosquitto_pub -t demo/one -h 129.151.125.171 -u $MOSQUITTO_USER -P $MOSQUITTO_PASSWORD -p 80 -m '{"message": 2}'
mosquitto_pub -t demo/one -h 129.151.125.171 -u $MOSQUITTO_USER -P $MOSQUITTO_PASSWORD -p 80 -m '{"message": 3}'
mosquitto_pub -t demo/one -h 129.151.125.171 -u $MOSQUITTO_USER -P $MOSQUITTO_PASSWORD -p 80 -m '{"message": 4}'
```


Sub: 

```shell
mosquitto_sub -t demo/one -h 129.151.125.171 -u $MOSQUITTO_USER -P $MOSQUITTO_PASSWORD -p 80
{"message": 1}
{"message": 2}
{"message": 3}
{"message": 4}
```


Sample Python Client

```shell
import paho.mqtt.client  as mqtt #import the client
import time
import ssl
​
############
​
def  on_message(client, userdata, message):
  print("message received ", str(message.payload.decode("utf-8")))
  print("message topic=", message.topic)
  print("message qos=", message.qos)
  print("message retain flag=", message.retain)
def  on_log(client, userdata, level, buf):
  print("log: ", buf)
  
############
​
broker_address =  "lb.yourdomain.com"
broker_port =  8883
print("creating new instance")
client = mqtt.Client("MQTT_Client",  transport='tcp')  # create new instance
client.on_message  = on_message # attach function to callback
client.on_log  = on_log
​
client.username_pw_set("mqttuser",  password="test")
client.tls_set(cert_reqs=ssl.CERT_NONE,tls_version=ssl.PROTOCOL_TLSv1_2, ciphers=None)
​
############
​
client.connect(broker_address, broker_port)  # connect to broker
client.loop_start()  # start the loop
print("Subscribing to topic",  "test/topic")
client.subscribe("test/topic")
print("Publishing message to topic",  "test/topic")
client.publish("test/topic",  "Test message")
time.sleep(5)  # wait
client.loop_stop()  # stop the loop
```