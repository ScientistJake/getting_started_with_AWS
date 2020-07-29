# Intro to cloud computing using AWS 
By Jake Warner and Jeff Brown
UNCW center for bioinformatics
UNCW DataScience

## Contents: 
* [Introduction](https://github.com/ScientistJake/getting_started_with_AWS#introduction)  
* [Getting started](https://github.com/ScientistJake/getting_started_with_AWS#getting-started)  
* [Accessing an instance via ssh](https://github.com/ScientistJake/getting_started_with_AWS#accessing-an-instance-via-ssh) 
     - [From Mac OSX / Linux](https://github.com/ScientistJake/getting_started_with_AWS#accessing-an-instance-via-ssh-from-a-mac-linux) 
     - [From Windows](https://github.com/ScientistJake/getting_started_with_AWS#accessing-your-instance-from-a-windows-machine) 
* [Playing around with your instance](https://github.com/ScientistJake/getting_started_with_AWS#playing-around-with-your-instance) 
* [Moving files on and off via scp](https://github.com/ScientistJake/getting_started_with_AWS#moving-files-on-and-off-via-scp) 
* [Monitoring the instance](https://github.com/ScientistJake/getting_started_with_AWS#monitoring-the-instance) 
* [Shutting down instances and cleaning up](https://github.com/ScientistJake/getting_started_with_AWS#shutting-down-an-instance-and-cleaning-up) 
* [Learn more](https://github.com/ScientistJake/getting_started_with_AWS#learn-more) 

## Introduction:  
If you're on this page, you've likely been tasked with getting an analysis pipeline, web server, or ther such utility onto the cloud. With that in mind, I'll dispense with the spiel about why cloud computing is useful and yadda yadda yadda. This guide covers AWS and is intended to make the transition to the cloud as painless as possible even if you know very little linux. If you're a bioinformaticien, you might also check out [jetsream-cloud](https://jetstream-cloud.org/) (stay tuned for a tutorial on that one).

## Getting started:  
First things first. If you don't have an Amazon ec2 account now's the time to register. Note that they have a [free tier](https://aws.amazon.com/free/) (which is just barrrrely enough to get a web server running fyi...) and an [Educate](https://aws.amazon.com/education/awseducate/) program.

Amazon offers a variety of [virtual machine configurations](https://aws.amazon.com/ec2/instance-types/), termed instances, with different cpu, memory, and storage configurations. You'll likely use a `T2` or `C5` instance if you're like me. In addition to these there are a variety of options for [storage volumes](https://aws.amazon.com/ebs/volume-types/) that can be mounted to an instance.  

For the sake of this tutorial we will walkthrough the spin up of a t2.micro instance (free tier eligible).

Once you log in to Amazon ec2, click 'Launch a virtual machine':  
![getting_started_1](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/getting_started_1.png?raw=true)

On the next page, you will be prompted to select an Image, which is essentially a collection of software that comes pre-installed on the instance. You can make and save your own image FYI for routine work, and there many community images that include all the useful stuff like anaconda, python, R, etc. For this tutorial we will just start a basic ubuntu image:   
![getting_started_2](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/getting_started_2.png?raw=true)

On the next step, choose the instance type. [Check this](https://aws.amazon.com/ec2/instance-types/) for a description of your options.  
![getting_started_3](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/getting_started_3.png?raw=true)

On this step you can add additional storage. [Check this](https://aws.amazon.com/ebs/volume-types/) for storage options.  
![getting_started_4](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/getting_started_4.png?raw=true)

Click through to the security step. This one is important. Here you're going to want to white list your local computer's IP address so that you can log onto the instance. Note that if you have a dynamically assigned address, or if you are constantly logging in from different networks, you will have to update this white-listed IP each time your address changes. If you have a static IP (like for your work computer that lives at work) then you're good to go.  
![getting_started_5](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/getting_started_5.png?raw=true)

The last step is to create a secure keyfile which has the extension `.pem`. This file is used for authentication when you sign in via ssh (more on that below). You can choose an existing key or create a new one. Here, we're creating a file called `UNCW_DS_AWSDEMO.pem`. Click download and don't forget about it.  
![getting_started_6](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/getting_started_6.png?raw=true)

Click launch instance and you're all set. On the ec2 console you should see it. Wait until the instance state says 'running', note the IP address and sign in (next section):  
![getting_started_7](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/getting_started_7.png?raw=true)

## Accessing an instance via ssh  

### Accessing an instance via ssh from a mac/ linux:  
First we need to make sure that ssh is installed an working properly on your machine. Linux and Mac OSX come preshipped with ssh so this is likely just a formality. To ensure you have ssh open a terminal and type:
`ssh`  
If you have ssh you will see something like this:  
```
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]
           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]
           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]
           [-i identity_file] [-J [user@]host[:port]] [-L address]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] destination [command]
```
If for whatever reason you don't have ssh install it like so: 
```
sudo apt install openssh-client
```

Next we need to add the security key `.pem` file [we downloaded during the instance setup](). You can keep key files wherever you want but best practice is to keep them in `/Users/$USER/.ssh/`. DO NOT KEEP THEM IN A GIT REPOSITORY because then your key file is visible to everyone who has access to your reposityory.  

Here I'm assuming it's sitting in your downloads folder. In your terminal execute the following commands, modifying them to suit your environment:  
```
# move the keyfile to a safe place
mv /Users/$USER/Downloads/UNCW_DS_AWSDEMO.pem /Users/$USER/.ssh/UNCW_DS_AWSDEMO.pem

# change the permissions for extra security
chmod 700 /Users/$USER/.ssh/UNCW_DS_AWSDEMO.pem

#add this to your ssh keychain
ssh-add /Users/$USER/.ssh/UNCW_DS_AWSDEMO.pem
``` 

A reminder that you can re-use keys for multiple instances. Just make sure that the instance security group is configured with an existing keypair.  

Now we're ready to log in. Unless you started a custom AWS instance that included usergroups your username will be the following:
| Instance      | Username |
| ----------- | ----------- |
| Amazon Linux | ec2-user |
| RHEL5 | root or ec2-user |
| Ubuntu | ubuntu |
| Fedora | fedora or ec2-user |
| SUSE | root or ec2-user |

```
#log in. The IP address is the public address of the machine
ssh ubuntu@3.12.196.125
```
This will show then:
```
The authenticity of host '3.12.196.125 (3.12.196.125)' can't be established.
ECDSA key fingerprint is XXXXXXxxxxx.
Are you sure you want to continue connecting (yes/no)?
```
Type `yes` and you're in:
```
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-1023-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Jul 28 18:53:01 UTC 2020

  System load:  0.0               Processes:           90
  Usage of /:   13.8% of 7.69GB   Users logged in:     0
  Memory usage: 16%               IP address for eth0: 172.31.26.171
  Swap usage:   0%

0 packages can be updated.
0 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
ubuntu@ip-172-31-26-171:~$
```

If you're having trouble loggin in, make sure that your whitelisted IP address on the AWS console matches the IP address of your local machine. This is really important if you're logging in from different coffee shops or something and your IP address is constantly changing!


If there is a problem with your key authentication, you can try to explicitely include it in your ssh call:  
```
ssh -i /path/my-key-pair.pem ubuntu@my-instance-public-dns-name
```  

### Accessing your instance from a windows machine:  
Use [Putty](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html)
Maybe someone with a PC can write this section.

## Playing around with your instance:  
For the section we will assume very little knowledge of Linux operating systems. First let's just explore our environment. Your login directory is the home directory for the user you logged in as:  
```
pwd
/home/ubuntu
```
The IP address is same as you would see on your AWS console and is listed after the `@` on your shell prompt. If you want to recover it and pipe it into something else you can use:
```
hostname -I | awk '{print $1}'
172.31.26.171
```

Our 8G of storage is mounted and available as well:  
```
df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            468M     0  468M   0% /dev
tmpfs            98M  748K   98M   1% /run
/dev/xvda1      7.7G  1.1G  6.6G  15% /
tmpfs           490M     0  490M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           490M     0  490M   0% /sys/fs/cgroup
/dev/loop0       98M   98M     0 100% /snap/core/9289
/dev/loop1       18M   18M     0 100% /snap/amazon-ssm-agent/1566
tmpfs            98M     0   98M   0% /run/user/1000
```

If the instance includes `htop` you can also use that to check out the instance.  
![htop](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/htop.png?raw=true)

I won't go through the details of a linux file system but at this point you would create a working directory and start computing:
```
mkdir analysis1
cd analysis1
python do_awesome.py
```

Oops we haven't uploaded do_awesome.py... Transfer it with the next step:  

## Moving files on and off via scp:  

From a mac / linux machine you can use the `scp` command. This utility should be included in most mac / linux distributions. If it's not you can install it using:
```
sudo apt install openssh-client
```

From your local machine, you can use scp to transfer files back and forth using the following syntax:
```
scp target_file destination_file
```
This will copy the `target_file` as the `destination_file`. When refering to files on the ec2 instance, you will need to supply the username, server address, and full path to the file so scp knows exactly where to place it. For example this command will transfer `do_awesome.py` from the desktop of our local machine to the ec2 instance:
```
scp /Users/$USER/Desktop/do_awesome.py ubuntu@3.12.196.125:/home/ubuntu/analysis1/do_awesome.py
```
The syntax for the server address is the `username` followed by `@` followed by the IP address, followed by a `:` and then the full path to wherever you want the file to be moved to including the filename. Again, you might have to specify the path to your keyfile with the `-i` option if you haven't added it to your ssh profile:
```
scp -i /Users/$USER/.ssh/UNCW_DS_AWSDEMO.pem /Users/$USER/Desktop/do_awesome.py ubuntu@3.12.196.125:/home/ubuntu/analysis1/do_awesome.py
```

To move files off, simply specify the remote file as the target and a local directory as the destination:
```
scp ubuntu@3.12.196.125:/home/ubuntu/analysis1/do_awesome.py do_awesome.py
```

Use the -r option to move entire directories back and forth. This will move the directory 'scripts' from the server to the local working directory:
```
scp -r ubuntu@3.12.196.125:/home/ubuntu/analysis1/scripts scripts
```

Note: For really big transfers, use [rsync](https://linux.die.net/man/1/rsync) since it's more flexible with network interrupts.

## Monitoring the instance:  
You can monitor the instance usage directly from the ec2 console under the "Monitoring" tab:  
![monitor](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/monitor.png?raw=true

From the instance itself you can monitor with htop if it's installed:
```
htop
```  
![htop](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/htop.png?raw=true)

Another good way is to use the [mail utility](https://www.binarytides.com/linux-mailx-command/) to send you an alert when your job is finished, or to periodically forward you a log file. This needs installing and a quick setup. Once installed you can for example add this to the end of a script and it will send you a 'job complete' notification so you can shut down your instance:
```
python long_running_script.py &&
echo "Job is complete" | mail -s "Notification from ec2 instance" drJake@example.com
```

## Shutting down an instance and cleaning up:

Shutting down your instance is super simple. Just transfer all your data off and shut it down from the console. You can stop the isntance or terminate. When you stop the instance it will shut it down and 'freeze' it. You won't be charged for computation time, because no computations are running, and you can resume it at anytime to get back all your data or programs you've installed. Note that restarting an instance will change its IP address. Terminating an instance on the otherhand removes it completely. The difference is that terminating an instance will also delete any attached bootable EBS volumes, ceasing any charges associated with those. If you don't know what these volumes are you can [read more here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html).   
![shutdown](https://github.com/ScientistJake/getting_started_with_AWS/blob/master/img/shutdown.png?raw=true)

## Learn more:

This is just an introduction. In follow up posts we will cover more advanced topics like automating this process using terraform and taking advantage of distributed computing.
