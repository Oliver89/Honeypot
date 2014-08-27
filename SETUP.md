AWS HONEYPOT SETUP
------------------

1. A functioning AWS EC2 instance is set up, in this case using an Amazon Linux instance

ENABLING PASSWORD AUTHENTICATION
--------------------------------

NOTE: This is significantly less secure than key-based authentication, and as such it is not
recommended for production servers. It is being used in this project to see what attacks will
be attracted.

On the EC2 instance, execute the following commands, thus backing up your original sshd_config:

	sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

	sudo vim /etc/ssh/sshd_config

Find the line that reads
	
	PasswordAuthentication no

and change it to 

	PasswordAuthentication yes

The configuration having been changed, restart the ssh session:

	sudo /etc/init.d/sshd restart

Test this by opening another SSH session to the server in a different terminal window. You should be
prompted for a password. Ctrl+C out of that, and return to your original SSH session.

INSTALLING COMPILERS
--------------------

The Amazon Linux instance is based on RHEL 5.0, and it does not include a compiler. In order to capture passwords,
we need a compiler to re-compile the sshd.

Execute the following command:

	sudo yum install gcc gcc-c++ autoconf automake zlib-devel openssl-devel make -y

COMPILING OPENSSH FROM SOURCE
-----------------------------

In the SSH session, execute the following commands in order to obtain and compile OpenSSH form source:

	cd

	mkdir ssh-source && cd ssh-source

	wget http://mirror.mcs.anl.gov/openssh/portable/openssh-5.8p1.tar.gz

	tar -xvzf openssh-5.8p1.tar.gz

	cd openssh-5.8p1

	./configure

	make

CONFIGURE THE LISTENING PORT
----------------------------

Since an instance of sshd is already running on your open AWS machine, and two processes cannot be bound to the same port,
in order to test the new sshd, it must be configured to run on a different port.

Execute the following commands:

	sudo cp sshd_config sshd_config.bak

	sudo vim sshd_config

Find the line that currently says

	#Port 22

and edit that line to

	Port 2222

Save and exit the file, then prepare the sshd for use:

	sudo cp sshd_config /usr/local/etc

	sudo mkdir /usr/local/etc/ssh

	sudo ssh-keygen -t dsa -f /usr/local/etc/ssh/ssh_host_dsa_key

Press return twice to use a blank passphrase when prompted.

STARTING SSHD
-------------

Execute the following commands to start sshd:

	sudo cp /usr/local/etc/ssh/ssh_host_dsa_key /usr/local/etc
	
	sudo cp /home/ec2-user/ssh-source/openssh-5.8p1/sshd

You will see a message saying "Could not load host key: /use/loca/etc/ssh_host_rsa_key", but that does not stop the sshd from
running.

Exectue the following to verify that sshd is listening on port 2222:

	netstat -an | more

You should see a LISTENING process on port 2222, and another on port 22.

The process listening on 2222 is the sshd that has just been compiled, and the process listening on port 22 is the built-in sshd Amazon
placed in the virtual machine.

ADJUSTING AWS FIREWALL
----------------------

Open a browser and go to 

https://console.aws.amazon.com/ec2

Log in with your Amazon account. From the dropdown in the top let-hand corner, select "EC2", then "Instances" from the menu on the left.
The category "Security Groups" will be visible in the ribbon above your instance list. Note which security group in which the instance exists.

In the panel on the left, select "Security Groups", and (unless the security group is being used for other instances), select the security group
relevant to the instance you wish to run as a honeypot. In the lower panel, select the "Inbound" tab. 
