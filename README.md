# shelly-mqtt-homeassistant
Reducing Shelly Scan Interval from 5 to 1 Second and Forwarding to Home Assistant via MQTT



**The current example applies to my devices. Logically, your device ID may be different, and the version or generation could also vary. You will need to manually edit these values for your setup.**

Used devices:

#Shelly Pro3EM,  
#Home Assistant,  
#MQTT Home Assistant Integration,  
#Mosquitto Broker Home Assistant Add-on,  
#MQTT Explorer Home Assistant Add-on,  

To make this work, you need to create a script on the Shelly device that updates the data every second and sends it to Home Assistant via MQTT. Special thanks to the JavaScript author turrican944 (Home Assistant Community).

```
// script by turrican944
let SHELLY_ID = undefined;
Shelly.call("Mqtt.GetConfig", "", function (res, err_code, err_msg, ud) {
  SHELLY_ID = res["topic_prefix"];
});

function timerHandler(user_data)
{
  let em = Shelly.getComponentStatus("em", 0);
         MQTT.publish(SHELLY_ID + "/status/em:0",JSON.stringify(em),0,false);
 }
Timer.set(3000, true, timerHandler, null);
```
![image](https://github.com/user-attachments/assets/6be20601-1476-4a19-bf0f-217b6e7a9e5b)



A working MQTT connection between the Shelly device and Home Assistant is required, and there are plenty of guides available for this, it's not complicated. Once that’s set up, use MQTT Explorer to find your Shelly device's ID and the exact topic.
![image](https://github.com/user-attachments/assets/47a1ede6-2cd2-4fa8-a2e7-1be5fd4dbc50)


After replacing the values in the script with your own and adding it to Home Assistant configuration.yaml, it should look like this:
```
mqtt:
  sensor:
     
   - name: "Shelly mqtt Active Power 1 sec"
     state_topic: "shellypro3em-YOUR-ID/status2/em:0"
     value_template: "{{ value_json.total_act_power | float | int }}"
     unit_of_measurement: "Wh"
     qos: 0
     force_update: true
     device_class: power
     state_class: measurement
```
![image](https://github.com/user-attachments/assets/bf1e9ffb-a3ee-4428-b546-7fdd99c38337)

     
**The new sensor will require a Home Assistant restart to appear. Before restarting, verify it under Developer Tools.**
