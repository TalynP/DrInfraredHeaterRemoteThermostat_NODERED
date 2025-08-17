# Dr-Infrared-Heater-Remote-Thermostat

Controls a Dr Infared Heater ([DR-996](https://drheaterusa.com/products/dr966-240-volt-hardwired-shop-garage-commercial-heater-3000-watt-6000-watt)) over a local Wi-Fi network.

This project uses a [Raspberry Pi Zero 2W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/) to host
[Node-RED](https://nodered.org/) to build the logic of the thermostat.  
Along with this a relay is used to control the heater to turn it on or off depening on room temperture.

Bill of Materials](./ReadMe Assets/Dr-Infrared-Heater-Remote-Thermostat BOM.xlsx) located in ReadMeAssets.

# Setup
### Raspberry Pi
Install Raspberry Pi OS using [Raspberry Pi Imager](https://www.raspberrypi.com/software/) on to Mirco SD card.
1. Update and Upgrade OS
```bash
sudo apt update && sudo apt upgrade -y
```  
2. [Install Node-RED](https://nodered.org/docs/getting-started/raspberrypi)  
```bash
bash <(curl -sL https://github.com/node-red/linux-installers/releases/latest/download/update-nodejs-and-nodered-deb)
```  
3. Enable Node-RED Service to run on boot  
```bash
sudo systemctl enable nodered.service
```  
4. Start Node-RED  
```bash
node-red-start
```  

### Node-RED Code and Access
1. Find the IP address or Hostname
```bash
hostname && hostname -I
```  
2. Enter IP address or hostname at port 1880 (default Node-RED port) into a web browser  
```bash
http://<your ip or hostname>:1880
```  
3. Login to Node-RED website  
4. Install [npm](https://www.npmjs.com/) packages from the manage pallet. Install the following:  
[node-red-dashboard](https://flows.nodered.org/node/node-red-dashboard) and [node-red-contrib-sensor-ds18b20](https://flows.nodered.org/node/node-red-contrib-sensor-ds18b20)  
5. Import the [Flows.json](./Flows.json) file ([Node_RED importing guide](https://nodered.org/docs/user-guide/editor/workspace/import-export))  
6. Deploy the new flows  
7. Enter the following in to a web browser to access the user interface  
```bash
http://<your ip or hostname>/ui
```  