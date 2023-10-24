# Spacemesh-logs-management-for-Ubuntu
go-spacemesh users can save log files by adding logging to the end of the command 
./go-spacemesh --config ./node-config.json | tee -a ./node.log

By the time the log can grow so big that no application is able to open it. We will use Logrotate for automatic management of log files.

In Ubuntu Logrotate is installed by default, but in case if it's not installed:
sudo apt update
sudo apt install logrotate -y


Check if it works:
logrotate --version


The current version is 3.19.0

First we need to create a configuration file. For that file you can select any location you like, For easy management I'm selecting ~/spacemesh folder, where I keep all go-spacemesh related files.

nano /home/<user>/spacemesh/logrotate.conf


Then paste in the following configuration:

/home/<user>/spacemesh/*/*.log {
    weekly
    maxsize 20M
    missingok
    rotate 10
    compress
    dateext
    copytruncate
    notifempty
}


This configuration is looking for log files in subfolders of /spacemesh. For exaple I've got folders /spacemesh/smh11, /spacemesh/smh12 etc.<br />
You can adjust this path as you like, also you can add path to multiple nodes, for example:<br />
/home/<user>/spacemesh/node1/*.log ~/node10/*.log /mnt/disk/node10/*.log {<br />

-----------------------------------------
weekly - log files will be rotated once a week. "daily" is also useful if you've got a lot of logs<br />
maxsize - log files will be rotated when the size of file reaches specified amount, in my case I set 20MB. It will ignore "weekly" if file size hits 20MB within a week<br />
missingok - ignores the error if log file is missing<br />
rotate - keeping a set amount of files before removing them<br />
compress - compressing rotated logs<br />
dateext - adding a date stamp to the name of files<br />
copytruncate - makes a copy before compressing and then truncates original file, prevents the errors from the node<br />
notifempty - doesn't rotate the log if it's empty<br />
-----------------------------------------

Next step is to create a state file, where logrotate saves information about it run:
logrotate ~/spacemesh/logrotate.conf --state ~/spacemesh/logrotate.state


Check the content of created file:
cat ~/spacemesh/logrotate.state


You'll see something like this:

logrotate state -- version 2
"/home/stizerg/spacemesh/smh11/smh11.log" 2023-10-7-12:55:4

That means logrotate succesfully recognized the file.

Next step is to create a cron job
crontab -e

(select 1)

we edit the current user's cron jobs. It shoud open the text editor.
At the bottom of the file, add the following line:
0 * * * * /usr/sbin/logrotate /home/<user>/spacemesh/logrotate.conf --state /home/<user>/spacemesh/logrotate.state


This new line specifies that the cron job will be executed every hour
Save and close the modified file. You will observe the following output:
crontab: installing new crontab

Now your log rotation is set.
Feel free to modify as you like.

P.S. Replace <user> with your user name
