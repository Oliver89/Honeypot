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
