# Dr Infrared Heater Remote Thermostat

Controls a Dr Infrared Heater ([DR-996](https://drheaterusa.com/products/dr966-240-volt-hardwired-shop-garage-commercial-heater-3000-watt-6000-watt)) over a local Wi-Fi network.

This project uses a [Raspberry Pi Zero 2W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/) with Raspbian GNU/Linux 12 (bookworm) with [Node-RED](https://nodered.org/) to build the logic of the thermostat.  
Along with this a relay is used to control the heater to turn it on or off depending on the room temperature.  
  
## User Pages  
![Common User Pages](./ReadMeAssets/UserPages.png)

# Software Setup
## Raspberry Pi
Install the Raspberry Pi OS on to a micro SD card using the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) software.
1. SSH into your Raspberry Pi
2. Update and Upgrade OS
```bash
sudo apt update && sudo apt upgrade -y
```  
3. Install [Node-RED](https://nodered.org/docs/getting-started/raspberrypi)  
```bash
bash <(curl -sL https://github.com/node-red/linux-installers/releases/latest/download/update-nodejs-and-nodered-deb)
```  
4. When it asks "Would you like to install the Pi-specific nodes?" select yes
5. Enable Node-RED service to run on boot  
```bash
sudo systemctl enable nodered.service
```  
6. Start Node-RED  
```bash
node-red-start
```  
7. Once you see "Started flows" type Ctrl+C
8. Enable the 1-wire interface
```bash
sudo raspi-config
```  
9. Navigate to Interface Options -> 1-Wire then select yes to enable
10. Select finish and reboot the system

## Node-RED
### RPi.GPIO Library Install
Depending on your Raspberry Pi OS version you may need to install/update the [RPi.GPIO](https://pypi.org/project/RPi.GPIO/) library. Command to install:
```bash
sudo apt install python3-rpi.gpio
```

Reboot your system to refresh everything once installed:
```bash
sudo reboot
```

### Access and Flows
1. Find the IP address or hostname
```bash
hostname && hostname -I
```  
2. Enter IP address or hostname at port 1880 (default Node-RED port) into a web browser  
```bash
http://<your IP or hostname>:1880
```  
3. Login to the Node-RED website  
4. Install [npm](https://www.npmjs.com/) packages from the manage pallet. Install the following:  
[flowfuse node-red-dashboard](https://flows.nodered.org/node/@flowfuse/node-red-dashboard) and [node-red-contrib-sensor-ds18b20](https://flows.nodered.org/node/node-red-contrib-sensor-ds18b20)  
5. Import the [Flows.json](./Flows.json) file ([Node_RED importing guide](https://nodered.org/docs/user-guide/editor/workspace/import-export))  
6. Deploy the new flows  
7. Enter the following in to a web browser to access the user interface
```
http://<your IP or hostname>:1880/dashboard/
```
8. The Minimum Room Temp page is hidden by default from the navigation bar to access this page go to
```
http://<your IP or hostname>:1880/dashboard/minroomtemp
```

### Flow Files
The Node-RED flows maintain two files the HeaterSchedule.json and HeaterState.log. The HeaterSchedule.json holds the time periods for the heater to maintain a temperature. Then the HeaterState.log writes when the heater turns on and off and logs the source that caused the heater to change state.  
  
An [example heater schedule](./ReadMeAssets/ExampleHeaterSchedule.json) is located in the ReadMeAssets and is described below:  
Sunday - The system will hold 70°F from 8am to 10am  
Monday - The system will hold 65°F from 9am to 11am and will hold 77°F from 2pm to 3pm  
Tuesday, Wednesday, Thursday, Friday, Saturday - No time periods


### Flow Details
For more information on how the Node-RED flows work reference [Node-REDFlowsExplained](./ReadMeAssets/Node-REDFlowsExplained.pdf) located in the ReadMeAssets.

## nginx and Remote Access
### nginx Reverse Proxy (Optional)
[nginx](https://nginx.org/) is used as a simple reverse proxy to allow a hostname such as `http://<your IP or hostname>.local` to show the Node-RED user interface (ui) dashboard website which defaults to `http://<your IP or hostname>.local:1880/dashboard/` and makes the user experience seamless. Below are the steps of how to setup nginx.  
1. Install nginx
```bash
sudo apt update
sudo apt install nginx -y
```
2. Create Node-RED site
```bash
sudo nano /etc/nginx/sites-available/nodered
```
3. Paste the contents below and change __XXXXX__ to your own hostname
```
server {
    listen 80;
    server_name XXXXX.local;

    # Redirect root to /dashboard
    location = / {
        return 301 /dashboard;
    }

    # Proxy /dashboard to Node-RED
    location /dashboard/ {
        proxy_pass http://127.0.0.1:1880/dashboard/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
4. Once pasted and hostname added type ctrl+O then enter to save the file
5. Once saved exit by typing ctrl+X
6. Point to Node-RED site by linking
```bash
sudo ln -s /etc/nginx/sites-available/nodered /etc/nginx/sites-enabled/
```
7. Remove default nginx site from sites enabled folder (optional but recommended)
```bash
sudo rm /etc/nginx/sites-enabled/default
```
8. Test and reload nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```
9. Navigate to your `http://XXXXX.local` (server name you put in at step 3)
  
Note - nginx steps assisted in creation with OpenAI ChatGPT.

### Remote Access to the Raspberry Pi (Optional)
A native feature for Raspberry Pi's is their [Raspberry Pi Connect](https://www.raspberrypi.com/software/connect/) software that allows secure remote access to your Raspberry Pi. Below I have steps to get this working using a headless (no desktop) Raspberry Pi but is best to reference the official Raspberry Pi Connect [documentation](https://www.raspberrypi.com/documentation/services/connect.html).  

1. Install Raspberry Pi Connect
```bash
sudo apt install rpi-connect-lite
```
2. Turn on
```bash
rpi-connect on
```
3. Enable user lingering to automatically re-login after reboot
```bash
loginctl enable-linger
```
4. Generate hyperlink to login to Raspberry Pi Connect and add your Pi
```bash
rpi-connect signin
```
5. Use the Raspberry Pi Connect website to remotely connect to your Pi  
# Hardware
## Bill of Materials (BOM)
A [bill of materials](./ReadMeAssets/DrInfraredHeaterRemoteThermostat_BOM.xlsx) has been provided and is located in the ReadMeAssets folder.

## Mounting Components
### Mounting to the Heater
The Raspberry Pi Zero 2W, 5v power supply and relays are mounted to the heater back mount plate then that is attached to the back of the Dr Infrared heater by two short self tapping screws. This mount was 3D printed with PETG (Polyethylene Terephthalate Glycol) the [HeaterBackMountPlate.stl](./ReadMeAssets/HeaterBackMountPlate.stl) file is located in the ReadMeAssets folder.  
  
Components mounted to heater back mount plate  
![Components mounted on back plate](./ReadMeAssets/ComponentsMountedOnBackPlate.jpg)  

Components and back mount plate mounted to the backside of the DR-996 heater
![Components mounted on back plate on backside of heater](./ReadMeAssets/ComponentsMountedToHeater.jpg) 


### Mounting the Temperature Probe
The DS18B20 temperature probe should be at ran as far as possible from the heater and about 60 inches (152cm) from the floor. The reason to put the probe as far as possible from the heater is to allow time for the heat in the room to reach the temperature probe and evenly warm the room. The mount for the temperature probe is screwed to the wall and was 3D printed with PETG the [TemperatureProbeCase.stl](./ReadMeAssets/TemperatureProbeCase.stl) file is located in the ReadMeAssets folder. 

Temperature probe mounted to the wall  

![Common User Pages](./ReadMeAssets/TemperatureProbeCase.jpg)

## Wiring
To wire the heater you should reference the [DR-966 Manual](./ReadMeAssets/DR-966Manual.pdf) in the ReadMeAssets folder or find it on the Dr Infrared Heater [website](https://drheaterusa.com/products/dr966-240-volt-hardwired-shop-garage-commercial-heater-3000-watt-6000-watt) for the DR-966. I have included a [wiring diagram](/ReadMeAssets/WiringDiagram.pdf) for how to wire the heater starting from the heavy gauge wires running from your circuit breaker panel to the heater. The wiring diagram also notes the wire gauges that I used. This is located in the ReadMeAssets folder.  

### Wiring the Temperature Probe
When wiring the temperature probe into the Raspberry Pi if you have already deployed the Node-RED flows you may need to go into the "Main" tab then find the red node labeled "Temp Probe" and select the sensor ID. If no ID shows up check your temperature probe wiring. Once the sensor ID is selected deploy the flows to see the live temperature displayed on the home ui page.

# Safety Disclaimer and Hazards
This project is great for learning about the Linux command line interface (CLI), Node-RED, 3D printing and the Internet of Things (IoT). Working with AC mains electricity, heating elements, and temperature controlled systems can be dangerous and may cause electrocution, burns, fire, property damage or violations of local electrical code if done incorrectly. __Always use safe practices for this project.__ The author assumes no responsibility, liability or fault for any damage, injury or other harm from the use of this project.

### Hazards  
- Electrical Hazards: electrocution, electrical shock, burns and fire  
- Heating Element Hazards: hot surfaces, burns and fire

### Preventing Hazards
- Check Component Ratings: Ensure components can handle the appropriate voltages and currents.
- Wire Inspection: Ensure wiring is properly insulated and no terminals are loose or exposed.
- __Hardware Shutoff__: Ensure you can easily and safely shutoff all power to the system, for example using a circuit breaker.
- __Test Test Test__: Never leave the system unattended when testing and monitor it closely.
- Follow local codes: Ensure your project complies with local electrical standards and regulations.

# Authors Note and Future Features
Finally, don't be dumb if you do not understand the dangers listed above don't do this project. I have done my best to test this system to make sure that the worst case scenario doesn't happen but cannot 100% guarantee the safety of this system.

### Future Features
- Bash script for install.
- Celsius option for UI.
- Custom PCB hardware documentation (KiCad).
- Room heat up time prediction using excel data analysis documentation.