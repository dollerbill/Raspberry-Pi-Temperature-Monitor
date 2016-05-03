#Raspberry Pi & DS18B20 temperature sensor with python


##Raspberry Pi
    I am running this script on a headless install of Raspbian Jessie Lite.

####Activating the temperature sensor
###Enable 1Wire devices
First edit the config file to enable 1Wire devices working through GPIO.

	sudo nano /etc/boot.txt
Add this line to the end of the file:

	dtoverlay=w1-gpio

###Enable necessary drivers

	sudo modprobe w1-gpio
	sudo modprobe w1-therm


###Test the thermometer
Cd into the 1Wire device folder and list files to make sure the thermometer is properly loaded and connected.

	cd /sys/bus/w1/devices/
	ls

There should be a foldering starting with 28-... Cd into that directory and check the output of the 1Wire slave file. (Make sure to use the full directory name listend after you ran the 'ls' command previously)

	cd 28-...
	cat w1_slave

You should see two lines similar to 
"f6 00 4b 46 7f ff 0c 10 7c : crc=7c YES
f6 00 4b 46 7f ff 0c 10 7c t=15375"
The first ending in YES tells us the thermomter is connected and working properly, the second shows us the temp (t) in thousandths of a degree Celsius. Divide this number by 1000 to see the current temperature (15375/1000 = 15.375* C). 


###Python script to display the temperature in Farenheit
Create a script that will display the temperature for us in degrees Farenheit

	sudo nano temperature.py



####ds18b20.service

	[Unit]
	Description=Run ds18b20 sensor

	[Service]
	User=your-username
	ExecStart=/usr/bin/env/python2 /your-path/temperature.py

#### Email the results
**Install SSMTP**

	sudo apt-get install ssmtp mailutils
Edit the ssmtp conf file
	
	sudo nano /etc/ssmtp/ssmtp.conf
Change the file to read:
	
	root=postmaster
	mailhub=smtp.gmail.com:587
	hostname=raspberrypi
	FromLineOverride=YES
	AuthUser=YourEmailAddress
	AuthPass=YourEmailPassword
	UseSTARTTLS=YES

In this example you must use a gmail account, otherwise you will need to change the mailhub line in the conf file.


**Create a Cron job to run the scrip daily and email the results**


	crontab -e
At the bottom add: 
	
	30 12 * * * python temperature.py | mail -s "Pool Temp" your.email@address.com
This will email the result of the script every day at 12:30pm. Make changes to the Cron schedule by editing the time or the asterisks, which mean "any."


