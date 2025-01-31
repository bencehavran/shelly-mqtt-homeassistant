# shelly-mqtt-homeassistant  
**Shelly MQTT Home Assistant Integration**  
Reducing Shelly Scan Interval from 5 to 1 Second and Forwarding to Home Assistant via MQTT  

    
**Introduction**  
This guide demonstrates how to reduce the scan interval of a Shelly device from 5 seconds to 1 second and forward the updated data to Home Assistant via MQTT.  

Please note that the example provided applies to my specific devices. Your device ID, version, and generation may differ, so you will need to manually adjust these values to match your setup.

**Required Devices & Tools**  
The following devices and tools are needed to complete this setup:

Shelly Pro 3EM  
Home Assistant  
MQTT Home Assistant Integration  
Mosquitto Broker Home Assistant Add-on  
MQTT Explorer Home Assistant Add-on  

You’ll need a working MQTT connection between the Shelly device and Home Assistant. There are many guides available online to help set this up—it's not complicated, so don’t worry if you’re new to it.  

**Creating the Script**   
To update the Shelly device data every second and send it to Home Assistant, you'll need to create a script on your Shelly device. Special thanks to **turrican944** from the Home Assistant Community for providing the JavaScript code that makes this possible!

Here’s the script:

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
Timer.set(1000, true, timerHandler, null);
```
![image](https://github.com/user-attachments/assets/6be20601-1476-4a19-bf0f-217b6e7a9e5b)


This script retrieves the Shelly device’s MQTT configuration and publishes the power data every seconds (you can change the interval if needed). Once set up, the data will be sent to Home Assistant via MQTT.

![image](https://github.com/user-attachments/assets/47a1ede6-2cd2-4fa8-a2e7-1be5fd4dbc50)


**Setting Up MQTT Integration in Home Assistant**
After the script is running on your Shelly device, you’ll need to configure Home Assistant to receive and process the data. Here’s how to set up an MQTT sensor in your configuration.yaml file:
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

**Important Notes:**  
Replace YOUR-ID: In the state_topic, replace YOUR-ID with the actual Shelly device ID. You can find this using MQTT Explorer.
Unit of Measurement: The unit is set to Wh (Watt-hour), but you can change it to W (Watt) if you’re measuring instantaneous power.
Force Update: Enabling force_update: true ensures that Home Assistant updates the sensor state even if the value hasn’t changed.


**Verifying and Restarting Home Assistant**  
After updating the configuration.yaml, the new sensor won’t appear immediately. You will need to restart Home Assistant for the new sensor to take effect.

Before restarting, you can verify that the sensor is recognized by checking the Developer Tools in Home Assistant. If everything looks good, restart Home Assistant to finalize the integration.

**Troubleshooting**  
If you’re experiencing issues with the sensor not appearing or updating, here are some common steps to troubleshoot:  

**Check MQTT Connection:** Ensure that Home Assistant is properly connected to your MQTT broker and that the Shelly device is successfully publishing to the correct topic.  
**Verify Topic**: Use MQTT Explorer to check that the Shelly device is sending data to the expected topic. The topic should match the one you’ve entered in configuration.yaml.  
**Restart Home Assistant**: If the new sensor isn’t showing up, try restarting Home Assistant and check the logs for any MQTT-related errors.  
