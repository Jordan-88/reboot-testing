# reboot-testing
Simple script I developed that would reboot devices in testing using bash.

Was brought into a new team at my employer and advised I needed to help test some devices.

These devices ran linux, I came into this project with little prior knowledge and no scripting experience.
We needed to ensure that these devices would be able to boot succesfully in a development environment and iron out any issues we discovered.

So with very minimal linux knowledge and zero bash scripting experience I set out on this task.

The first version of this script was simply an expect script that would ssh into the device and run the commands to reboot, gaining sudo and then intializing the reboot. Very simple and it did its job.

However one of the leads wanted logging, when the device failed, and how many reboots we performed.
This stumped me and thats when I realized I could launch my expect script in a subshell from a bash script.

What I then did was reinvent how the script launched.

###############################V1###################################
#This was version one of my script, very basic, and I had multiple loaded into my crontab.

#!/usr/bin/expect -f
#host 1 reboot script
spawn ssh user@10.10.10.1

expect "user@10.10.10.1's password:"
send "password\r"

sleep 0.2

send "sudo reboot\r"

expect "password:"
send "password\r"

send "reboot\r"
expect eof
exit

##############################V2######################################

From here I included this new bash script. As you can see below, this one was much more advanced it had an if/then/else statment.
Basically in english it would ping the host ip if it got a response it would then initiate the expect reboot script and log the event with a timestamp to a log file. However if the device was down it would also log the failure and end.

#!/bin/bash

reboot=/home/user/rebootscripts/reboot_1.exp
faillog=/home/user/logs/hostdown.txt
rebootlog=/home/user/logs/rebootlog.txt

export reboot
export faillog
export rebootlog



(ping -c 1  10.10.10.1 > /dev/null 2>&1
        if [ $? -eq 0 ]
                then ( expect $reboot ) > /dev/null 2>&1 | echo "Reboot of HOST 1 complete $(date)" >> $rebootlog
                exit 0
        else echo "HOST 1 IS DOWN $(date)." >> $faillog

###############################################################################################

Because of the environment I was in I needed this to be modular and scalable and be able to run parallel. Thats where I added the final script that would be loaded in the cron tab as follows.

* * * * * /bin/bash /home/jordan/shellscripts/reboot/masterreboot.sh

From within the masterreboot.sh file I could add and remove devices as needed and simply copy and paste the code from the other scripts.
I also included a counter to see how many times the script ran.
###################################Masterboot##################################################

#!/bin/bash
#master reboot script, launches individual child scripts

reboot1=/home/user/shellscripts/reboot1.sh
reboot2=/home/user/shellscripts/reboot2.sh
reboot3=/home/user/shellscripts/reboot3.sh
reboot4=/home/user/shellscripts/reboot4.sh
reboot5=/home/user/shellscripts/reboot5.sh
reboot6=/home/user/shellscripts/reboot6.sh
reboot7=/home/user/shellscripts/reboot7.sh
reboot8=/home/user/shellscripts/reboot8.sh
reboot9=/home/user/shellscripts/reboot9.sh



export reboot1
export reboot2
export reboot3
export reboot4
export reboot5
export reboot6
export reboot7
export reboot8
export reboot9


(sh $reboot1) &
(sh $reboot2) &
(sh $reboot3) &
(sh $reboot4) &
(sh $reboot5) &
(sh $reboot6) &
(sh $reboot7) &
(sh $reboot8) &
(sh $reboot9) &


echo "Reboot script ran $(date)" >> /home/user/logs/rebootcount.txt

######################################################################################################

Now this was all 100% self taught / developed on the fly I have never created anything in any scripting language in the past.

It works great and does what is needed and I can scale it up to many more hosts if needed. Any input on this would be appreciated.
