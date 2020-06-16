
Note we have vagrant user with UID and GID as 1000
Inside containere by default jenkins image, jenkins user have UID and GID as 1000

vagrant and jenkins user have same UID and GID

# A. Set Up the NFS Server
from 10.0.0.5
========================
sudo apt update
sudo apt install nfs-kernel-server

sudo mkdir -p /srv/nfs4/data/jenkins/jenkins_home
sudo mkdir -p /srv/nfs4/data/jenkins/jenkins_db

sudo mkdir -p /opt/jenkins/data/jenkins_home
sudo mkdir -p /opt/jenkins/data/jenkins_db

sudo useradd -u 1001 -m jenkins
# we have vagrant user with UID and GID as 1000

sudo chown vagrant:vagrant /opt/jenkins/data/jenkins_home   // works

------------------------------
# note list user id - > sudo cat /etc/passwd
#vagrant@ubuntuvm01:~$ sudo cat /etc/passwd | grep 999
#vboxadd:x:999:1::/var/run/vboxadd:/bin/false

#vagrant@ubuntuvm01:~$ id vboxadd
#uid=999(vboxadd) gid=1(daemon) groups=1(daemon)

add user vboxadd to group vagrant 
	sudo usermod -a -G 1000 vboxadd 
	
	sudo usermod -a -G 999 vboxadd

vagrant@ubuntuvm01:~$ id vboxadd
uid=999(vboxadd) gid=1(daemon) groups=1(daemon),1000(vagrant)

vagrant@ubuntuvm01:~$ getent group | grep 999
vboxsf:x:999:

sudo usermod -g 999 vboxadd

--------------------------------------------------
sudo useradd -g jenkins 999
#uid=1003(999) gid=1001(jenkins) groups=1001(jenkins)



sudo usermod -a -G 485 vboxadd	

sudo chown -R vboxadd:vagrant /opt/jenkins/data/jenkins_db
sudo chmod 777 /opt/jenkins/data/jenkins_db

#vboxadd:vboxsf =999:999

#sudo chown -R nobody:nogroup /opt/jenkins/data/jenkins_db
#sudo chmod 777 /opt/jenkins/data/jenkins_db

#sudo chown vagrant:vagrant /opt/jenkins/data/jenkins_db


#sudo chown jenkins:jenkins /opt/jenkins/data/jenkins_home  // no works as jenkins have uid as 1001 which is not matching inside container . 
#sudo chown jenkins:jenkins /opt/jenkins/data/jenkins_db

#sudo chown -R nobody:nogroup /opt/jenkins/data/jenkins_home


sudo mount --bind /opt/jenkins/data/jenkins_home /srv/nfs4/data/jenkins/jenkins_home
sudo mount --bind /opt/jenkins/data/jenkins_db /srv/nfs4/data/jenkins/jenkins_db
sudo vi /etc/fstab
```
/opt/jenkins/data/jenkins_home     /srv/nfs4/data/jenkins/jenkins_home      none   bind   0   0
/opt/jenkins/data/jenkins_db     /srv/nfs4/data/jenkins/jenkins_db      none   bind   0   0
```

sudo vi /etc/exports
```
/srv/nfs4         10.0.0.0/24(rw,sync,no_subtree_check,crossmnt,fsid=0)
/srv/nfs4/data/jenkins/jenkins_home     10.0.0.6(rw,sync,no_subtree_check)
/srv/nfs4/data/jenkins/jenkins_home     10.0.0.7(rw,sync,no_subtree_check)

/srv/nfs4/data/jenkins/jenkins_db     10.0.0.6(rw,sync,no_subtree_check)
/srv/nfs4/data/jenkins/jenkins_db     10.0.0.7(rw,sync,no_subtree_check)
```

sudo exportfs -ra
#sudo exportfs -v

sudo ufw allow from 10.0.0.0/24 to any port nfs
sudo ufw enable
sudo ufw status

#In order for the users on the client machines to have access, NFS expects the client’s user and group ID’s to match with those on the server.

# B. Set Up the NFS Clients 
from 10.0.0.5 and from 10.0.0.6
========================
sudo apt update
sudo apt install nfs-common

sudo adduser -u 1001 jenkins

#sudo mkdir -p /opt/jenkins/data/jenkins_home
sudo mkdir -p /opt/jenkins/data
sudo mkdir -p /data/jenkins/jenkins_home
#sudo mkdir -p /opt/jenkins/data/jenkins_db
sudo mkdir -p /data/jenkins/jenkins_db


#sudo mount -t nfs -o vers=4 10.0.0.5:/data/jenkins/jenkins_home /opt/jenkins/data/jenkins_home
sudo mount -t nfs -o vers=4 10.0.0.5:/data/jenkins/jenkins_home /data/jenkins/jenkins_home
#sudo mount -t nfs -o vers=4 10.0.0.5:/data/jenkins/jenkins_db /opt/jenkins/data/jenkins_db
sudo mount -t nfs -o vers=4 10.0.0.5:/data/jenkins/jenkins_db /data/jenkins/jenkins_db
sudo vi /etc/fstab
10.0.0.5:/data/jenkins/jenkins_home /data/jenkins/jenkins_home        nfs   defaults,timeo=900,retrans=5,_netdev	0 0
10.0.0.5:/data/jenkins/jenkins_db /data/jenkins/jenkins_db       nfs   defaults,timeo=900,retrans=5,_netdev	0 0

df -h
du -sh /opt/jenkins/data

## Another option to mount the remote file systems is to use either the autofs tool or to create a systemd unit

symbolic link
#ln -svnf FILE LINK_NAME
sudo ln -svnf /data/jenkins/jenkins_home  /opt/jenkins/data/jenkins_home 
sudo ln -svnf /data/jenkins/jenkins_db  /opt/jenkins/data/jenkins_db

sudo chown  -h vagrant:vagrant /opt/jenkins/data/jenkins_home 
sudo chown  -h vagrant:vagrant /opt/jenkins/data/jenkins_db

# C. Testing NFS Access
from  10.0.0.5 and 10.0.0.6
========================

sudo touch /opt/jenkins/data/test.txt
touch: cannot touch '/opt/jenkins/data/test.txt': Permission denied

sudo useradd -u 1001 jenkins
sudo -u jenkins touch /opt/jenkins/data/test.txt

sudo touch /opt/jenkins/data/jenkins_home/a.txt

D. unmount commands
==========================================
sudo umount /opt/jenkins/data
sudo rm -R /opt/jenkins/data

#ref : https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-18-04

