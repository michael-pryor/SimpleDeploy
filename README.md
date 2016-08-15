# SimpleDeploy
## Purpose
I developed simple deploy over a weekend, as I wanted a way of monitoring and deploying to the servers of my startup without hassle. I couldn't find any simple, easy, non bloated and free solutions, so I wrote my own.

There is still plenty of work that could be done, so please feel free to contribute. This is tested on CentOS.

## Usage
### On the box
#### state
Running global state on any server setup with SimpleDeploy will yield for example:
```
[jobe@rabbit bin]$ global state
[STATE - commander (commander)] - is RUNNING (pid = 24138, uptime = 15:55:12)
[STATE - rabbit (governor)] - is RUNNING (pid = 24146, uptime = 15:55:12)
```

#### lesslog
Above we have two processes installed, one commander and one governor, named rabbit. These have been running for 15 hours each.

If I want to view the logs of rabbit, I simply run:
```
global lesslog rabbit
```

and immediately the log is opened in less.

#### start/stop/restart
I can restart rabbit:
```
[jobe@rabbit bin]$ global restart rabbit
[STOP - rabbit (governor)] - success
[START - rabbit (governor)] - started
```

Or I can restart all processes on the server:
```
[jobe@rabbit bin]$ global restart
[STOP - commander (commander)] - success
[START - commander (commander)] - started
[STOP - rabbit (governor)] - success
[START - rabbit (governor)] - started
```

I can do the same thing with stop and start.

### Scheduled jobs
For automatically maintaining your server and keeping things running, I suggest using cron. I've set mine up like this:
```
MAILTO="michael@michael.com"

# Every minute check processes are running, restart if necessary and send an email.
* * * * * source /home/jobe/.bashrc; global audit_regular

# Every day, send an email describing the state of the host and its jobs.
0 5 * * * source /home/jobe/.bashrc; global audit_daily

# Every Monday at 7am, archive the logs.
0 7 * * 1 source /home/jobe/.bashrc; global archive_logs
```

The scripts echo output expecting that output to be forwarded somewhere e.g. emailed to you.

In the above setup, I receive a daily email which looks like this:
```
[AUDIT_DAILY - commander (commander)] - [GREEN] - No recent automatic restarts detected, this is good
[AUDIT_DAILY - commander (commander)] - [GREEN] - is RUNNING (pid = 9322, uptime = 07:51:09)
[AUDIT_DAILY - rabbit (governor)] - [GREEN] - No recent automatic restarts detected, this is good
[AUDIT_DAILY - rabbit (governor)] - [GREEN] - is RUNNING (pid = 9344, uptime = 07:51:09)

Disk space stats
Use%
  1%
```

and if a process crashes it is automatically restarted, and on the first crash, I get a more concerning email.

### Deployment
SimpleDeploy gives you the ability to run one command from your development machine, and deploy and restart on all hosts in your estate.

For example:
```
./deploy -hf hosts/examplehosts -i
```

where hosts/examplehosts contains:
```
myserver.domain.com
myotherserver.domain.com
```

This will deploy the latest package (by modification time) in SimpleDeploy/builds to both servers. Because of the 'i' flag, it will then install the package on both hosts. Installing involves extracting the archive and restarting processes.

## Setup
### Setup the ~/.bashrc
Parts of the script will source ~/.bashrc, so this needs to be setup correctly on any host that you deploy to. You need to include the following variables:
- **LOG_DIR**: Full path to where log files should be stored. All standard output and standard error is forwarded to a log file in this directory.
- **PACKAGE_DIR**: When using the deploy script, directory where packages should be copied to.
- **PROJECT_DIR**:Root directory where archives are extracted to. This directory must also include bin at $PROJECT_DIR/bin, where bin is the folder containing the scripts of SimpleDeploy. I recommend that your packaging script packages in bin at the root of the archive, so that when extracted it will push the latest version automatically.
- **PATH**: I recommend that you include the bin directory in your path, so that you can run "global" from any directory.

### Setup your SSH keys
You want to be able to deploy without typing in passwords, so I suggest setting up SSH keys from your deployment machine onto the servers of your estate.

### Setup scheduled jobs
Setup your scheduled jobs (I use cron), this is explained in the "Scheduled Jobs" section of "Usage".

### What do I need to do?
This should work out of the box, all you need to do is make a packaging script to build the .tgz deploy package for your application.