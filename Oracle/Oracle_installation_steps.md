## Oracle installation on EC2 instance

This YouTube video follows the workflow I’ve created below –
[![Alt text]](https://www.youtube.com/watch?v=Vcdf8qSndOs&list=PLqFI2r42bxjI_m2tUH18dT15wAu9o0uSV)

1.	Download MobaXterm
2.	Set up Ec2 instance using OL7.2-x86_64-HVM-2015-12-10 Amazon Machine Image (AMI) with three additional volumes of 5, 8 and 10 gb (this will make your life easier).
3.	Login via MobaXterm as ec2-user and change password
•	sudo passwd ec2-user
4.	Install packages needed for instance
•	sudo yum install wget zip unzip -y
•	sudo yum install perl-libwww-perl.noarch -y
•	sudo yum install oracle-rdbms-server-12cR1-preinstall -y
5.	Mount volumes, format disks, mkdir
1.	sudo mkfs -t ext4 /dev/xvdb (be super careful here)
2.	sudo mkfs -t ext4 /dev/xvdc (be super careful here)
3.	sudo mkdir -p /swapfile1
4.	sudo mount /dev/xvdb /swapfile1 (swap file)
5.	sudo mkdir -p /u01/software
6.	sudo mount /dev/xvdf /u01/software (zip files)
7.	sudo mkdir -p /u01/app/oracle/oradata/orcl
8.	sudo mount /dev/xvdc /u01/app/oracle/oradata/orcl (data files)
6.	Add swap file
•	sudo dd if=/dev/zero of=/swapfile1/swapfile  bs=1024 count=3145728 (creates 3G swapfile)
•	sudo chown root:root /swapfile1/swapfile
•	sudo chmod 0600 /swapfile1/swapfile
•	sudo  mkswap /swapfile1/swapfile
•	sudo swapon /swapfile1/swapfile
•	free -m
7.	Make fstab entries
•	lsblk (gives names of volumes)
•	sudo cp /etc/fstab /etc/fstab.orig (back up fstab)
•	sudo vi /etc/fstab
•	/swapfile1 none swap sw 0 0
•	/dev/xvdc /u01/app/oracle/oradata/orcl ext4 defaults 0 0
•	/dev/xvdf /u01/software ext4 defaults 0 0
•	sudo mount -a (remount everything to make sure it worked)
8.	 Give oracle user ownership of directories
•	sudo chown -R oracle.oinstall /u01/app/oracle/oradata/orcl
•	sudo chown -R oracle.oinstall  /u01/software
•	sudo chown -R oracle.oinstall /u01
9.	Change hostfile for instance by adding hostname to localhost
•	hostname
•	sudo vi /etc/hosts
10.	Change password for oracle user and make it possible for user to connect remotely (Managing User Accounts on Your Linux Instance)
•	sudo passwd oracle
•	su oracle (switch to oracle user)
•	cd ~(make sure your are in oracle user home)
•	mkdir .ssh (create location for key file)
•	chmod 700 .ssh (set permissions)
•	touch .ssh/authorized_keys (create file)
•	chmod 600 .ssh/authorized_keys (set permissions)
•	“GET http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key&gt;.ssh/authorized_keys” (copy public key to file)
•	log out and login as oracle user
11.	Upload database zip files into /u01/software (this takes a while)
12.	Unzip files
•	cd /u01/software
•	unzip linuxamd64_12102_database_1of2.zip
•	unzip linuxamd64_12102_database_2of2.zip
13.	Start install
•	cd /ora_software/database
•	./runInstaller
14.	Run scripts as root (careful here you need to open separate instance of MobaXterm)
•	sudo /u01/app/oraInventory/orainstRoot.sh
•	sudo /u01/app/oracle/product/12.1.0/dbhome_1/root.sh
15.	Say OK to run dbca to create a database
16.	Update tnsnames.ora, listener.ora files by replacing localhost with actual host name
•	cd /u01
•	find -name tnsnames.ora
•	find -name listener.ora
•	hostname
•	vi ./app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora
•	vi ./app/oracle/product/12.1.0/dbhome_1/network/admin/listener.ora
17.	Restart listener and database
•	. oraenv
•	lsnrctl stop
•	lsnrctl start
•	sqlplus sys as sysdba
•	shutdown immediate;
•	startup;
18.	At this point you should be able to connect to your database remotely with SQL Developer or your favorite tool, remember if you are shutting down your EC2 instance to first stop the listener and the database.

