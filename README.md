# Raspberry Pi & DS18B20 temperature sensor with python
Monitor and log temperatures using python, a Raspberry Pi and a wired temperature sensor. 

## Raspberry Pi
I am running this script on a headless install of Raspbian Jessie Lite. The DS18B20 sensor is attached to GPIO pins 1(3.3v), 6(Ground), and 7(GPIO4/Data) and is pulled up with a 4.7kOhm resisitor.

## Activating the temperature sensor
### Enable 1Wire devices
First edit the config file to enable 1Wire devices working through GPIO.

	sudo nano /etc/boot.txt
Add this line to the end of the file:

	dtoverlay=w1-gpio

### Enable necessary drivers

	sudo modprobe w1-gpio
	sudo modprobe w1-therm


### Test the thermometer
Change into the 1Wire device folder and list files to make sure the thermometer is properly loaded and connected.

	cd /sys/bus/w1/devices/
	ls

There should be a foldering starting with 28-... Change into that directory and check the output of the 1Wire slave file. (Make sure to use the full directory name listend after you ran the 'ls' command previously)

	cd 28-...
	cat w1_slave

You should see two lines similar to 
	
	f6 00 4b 46 7f ff 0c 10 7c : crc=7c YES
	f6 00 4b 46 7f ff 0c 10 7c t=15375
The first line ending in YES tells us the thermomter is connected and working properly, the second shows us the temp (t) in Celsius in thousandths of a degree. Divide this number by 1000 to see the current temperature (15375/1000 = 15.375* C). 


### Python script to display the temperature in Farenheit
Move the temperature.py script to your Raspberry Pi. Make note of the location because we'll be using it later to check our results.
Line 58 of the script:

	return degrees_f
	
Can be changed between 'c' and 'f', depending which temperature scale you prefer your results in.
To test script output from the sensor, change into the director containing the python script and run:

	python temperature.py
This will return your current temperature value in your specified temperature scale.


## Email the results
### Install SSMTP

	sudo apt-get install ssmtp mailutils
### Edit the ssmtp conf file
	
	sudo nano /etc/ssmtp/ssmtp.conf
Change the file to read:
	
	root=postmaster
	mailhub=smtp.gmail.com:587
	hostname=raspberrypi
	FromLineOverride=YES
	AuthUser={YourEmailAddress}
	AuthPass={YourEmailPassword}
	UseSTARTTLS=YES

In this example you must use a gmail account, otherwise you will need to change the mailhub line in the conf file.


### Create a Cron job to run the script daily and email the results


	crontab -e
At the bottom add this line, and remember to change it to match your specific values: 
	
	30 12 * * * python {/yourpath}/temperature.py | mail -s "Temp Reading" {your.email@address.com}
	
This will email the result of the script every day at 12:30pm. Changes can be made to the Cron schedule by editing the five positions, which are, in order: minute, hour, day of month, month, day of week. Leaving an asterisk in any position will set the value to any (e.g. an asterisk in the 4th position would trigger the event every month, at the timing of the remaining values).
