The payload is formatted in JSON notation. The measurement reference is the Unix date/time in milliseconds.

Currently only the weight of an user is transmitted. You could test the openScale sync MQTT support for example using [CloudMQTT](https://www.cloudmqtt.com/).

_Note: openScale supports only MQTT 3.1 **not** MQTT 5.x_

**Adding a measurement**

topic `openScaleSync/measurements/insert`

payload `{"date":1572082380000,"weight":80.0}`

**Update a measurement**

topic `openScaleSync/measurements/update`

payload `{"date":1572082380000,"weight":120.0}`

**Delete a measurement**

topic `openScaleSync/measurements/delete`

payload `{"date":1572082380000}`

**Clear all measurements**

topic `openScaleSync/measurements/clear`

payload `true`
