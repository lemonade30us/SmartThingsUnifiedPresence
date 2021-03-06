# SmartThings Unified Presence
A solution that looks at WiFi, Bluetooth or BLE signals to determine if a given person is present. Presence is marked in SmartThings device handler.

See the uploaded file Unified Presence V1.0 Installation Instructions.pdf for more complete instructions.

Overview: This project consists of two pieces. A Python script with supporting shell script that nominally runs on a Raspberry Pi. This script is configured with people, devices and device addresses to monitor for presence.  One person may have up to four devices. All the devices must be "out" for the person to be considered out. Any one device in and the person is considered in. The second piece is a device handler for SmartThings that receives the information via a JSON file and marks the persons presence accordingly in SmartThings.

I built this using a RPi 4B but it runs in production on an RPi Zero W. These steps are based on a fresh install (Buster) so may need to be adjusted based on your individual version.

#Network Prep\
You will need a static IP address in order for this to work reliably. You can either configure a static IP or use a DHCP reservation. While you are at it I'd recommend creating DHCP reservations for all of the IP devices that you plan to monitor, phones for example. 

Steps to get installing RPi with DeviceMon and WebServer services
#Personally, I like to use Microsoft Remote Desktop but it’s your choice. It’s not required but it makes the installation and editing of #the details in the Python script a little easier but you could use a terminal session and nano to edit the file.\
sudo apt-get install xrdp

#Gets everything up to snuff if not already done at install\
sudo apt-get update\
sudo apt-get upgrade

#Gets rid of excess junk\
sudo apt autoremove

#Tools for XWindows Manipulation - these are used by menu provided to set the size of the window.\
sudo apt-get install xdotool

#Download the services.tar.xz file and extract it to the services directory in /home/pi so the full path is /home/pi/services\
Change permissions on shell scripts /home/pi/services/menu.sh and /home/pi/service/devicemon/devicescan.sh to be executable.

#Bluetooth is probably already installed, but if not do this.\
#sudo apt-get install bluetooth\
#sudo apt-get install bluez

#Install the bluetooth libraries\
sudo apt-get install libbluetooth-dev

#Setup bluez for Python\
sudo apt-get install python-pip python-dev ipython\
sudo pip install pybluez\
sudo pip3 install pybluez

#Configure the services. This allows all of these to run in the background and the menu.sh makes it easy to stop\start\review each of the services without having to remember anything.  This also ensures that the services will start automatically at boot time. These command create a hard link a service file and then start that service.\
sudo ln /home/pi/services/devicemon/devicemon.service /etc/systemd/system\
sudo systemctl enable devicemon.service

sudo ln /home/pi/services/devicemon/btcscan.service /etc/systemd/system\
sudo systemctl enable btcscan.service

sudo ln /home/pi/services/devicemon/webserver.service /etc/systemd/system\
sudo systemctl enable webserver.service

#Now you have to configure the user names, device names and device addresses (Bluetooth\BLE and IP). All of this information is stored near the top of the script /home/pi/services/devicemon/devicemon.py and can be edited using the Thonny Python editor.  You can look at the other settings for scan frequency, timeouts and retries but I'd suggest leaving as is until you have confirmed the basic operation.

#Now you can start the menu system\
/home/pi/services/menu.sh

#From here you can start each of the services. Watch for any errors as they start.\
#Option 1 - Start Btcscan. This starts bluetoothctl in scan mode so that BT and BLE signals are captured.\
#Option 11 - Start Webserver. This starts a Python simple http server using port 8001 serving out the directory /home/pi/services/devicemon/webserverEnter your device and user specific information into the devicemon.py file.\ 
#Option 21 - Start Devicemon. This starts the devicemon.py script and should generate the output file /home/pi/services/devicemon/webserver/unifiedpresence.json. Point a browser to your Raspberry Pi port 8001. For example: http://192.168.0.110:8001 and you should see this file present. You can open the file and check its contents to make sure it is updating approximately once per minute.

#That is all that is required for now on the Pi device.  You can check on the status of the three services using options 4, 14 and 24 of the menu and they should all be running and you should be able to see the output for each of these. Press PgDn for more, q and then enter to exit. If you reboot the Pi all of these services will start automatically.

#Now it's time to configure the SmartThings device.\
Locate the code for the Unified Presence device handler in this repository and then "Create New Device Handler" in your SmartThings account. You can now create a new device using the Unified Presence device handler. Let's call it "!Gary Presence" for easy of reference.

On your SmartPhone open up SmartThings Classic and go to your list of Things. Open up "!Gary Presence" and click on the gear in the upper right corner. Here you must enter the following information: IP address for your Raspberry Pi, port (8000) and the name of the person this Presence Monitor will represent (Bob). This name must exactly match one of the names you entered in the Python script (it is case sensitive). Change the Auto refresh for now to 1 minute for testing, you can change it back later. If not connecting check the PDF for more detailed instructions.

You can now use this presence sensor in SmartThings in the normal way.

See Unified Presence V1.0 Installation Instructions.pdf for more detail including how to add multiple devices.
