#Raspberry Pi & DS18B20 temperature sensor with python


##Run on Raspberry Pi
    Running headless on an install of Jessie Lite

###I Used Systemd/Timers
#### ds18b20.timer

    [Unit]
    Description=Run ds18b20 for temperature

	[Timer]
	OnBootSec=1min
	OnUnitActiveSec=1min

	[Install]
	WantedBy=timers.target

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
Change the file so it says:
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


