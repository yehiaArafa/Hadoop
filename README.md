# Hadoop
This tutorial will get you familiar with installing and configuring Hadoop (multi-node intallation) with basic feature on 3 nodes.

#### Make sure you have installed 3 linux images on three different machines and they are up and running.
Say the first machine **"master"** is the master, while The second **"slave1"** and the third **"slave2"**, which are the slaves .
#### For simplicity and the sake of this tutorial will assume these IP adresses and their corresponding names for our three nodes:
**master:**  
IP   : 192.168.1.103  
HDFS : Namenode  
**slave1:**  
IP   : 192.168.1.104  
HDFS : Datanode  
**slave2:**  
IP   : 192.168.1.105  
HDFS : Datanode  

#### Follow these steps given below to have an HDFS between these 3 machines:

## Step 1
***Installing JAVA***   
Java is the main prerequisite for Hadoop. First of all, you should verify the existence of java in your system using:
```
java -version
```
If everything works fine it will give you the following output:
```
java version "1.7.0_71" 
Java(TM) SE Runtime Environment (build 1.7.0_71-b13) 
Java HotSpot(TM) Client VM (build 25.0-b02, mixed mode)
```
If java is not installed then install java:
```
sudo apt-get install default-jre
sudo apt-get install default-jdk
```
Set up **PATH** and **JAVA_HOME** variables, by add the following commands to **~/.bashrc** file:
```
export JAVA_HOME=/usr/local/jdk1.7.0_71
export PATH=$PATH:$JAVA_HOME/bin
```
## Step 2
**MAPPING THE NODES**   
with each other,You have to edit hosts file in **/etc/hosts** on all nodes, specify the IP address of each system followed by their host names, add these lines in **/etc/hosts**:
```
192.168.1.103  master
192.168.1.104  slave1
192.168.1.105  slave2
```
## Step 3
**CONFIGURING THE KEYS**   
generate the public/private keys in the master node and put the public key of the master in each of the slaves authorizedkey file:
```
ssh-keygen -t rsa 
```
This will generate a public and private key on the machine. Make sure to copy the public key located in:
```
~/.ssh/id_rsa.pub
```
And add that key to the authorized keys on datanodes machines, the authorized keys file will be located in:
```
~/.ssh/authorized_keys
```

## Step 4
**INSTALLING HADOOP ON MASTER**   
On the master node install hadoop from one of the stable <a href="http://apache.claz.org/hadoop/common/">mirrros</a>   
```
mkdir /opt/hadoop 
cd /opt/hadoop/ 
wget http://www-eu.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz 
tar -xzf hadoop-1.2.0.tar.gz 
mv hadoop-1.2.0 hadoop
chown -R hadoop /opt/hadoop 
cd /opt/hadoop/hadoop
```
Set $HADOOP variable by adding the following to **~/.bashrc**   
```
export HADOOP=/opt/hadoop/
export PATH=PATH:$HADOOP/bin
```
Load **./bashrc**
```
source ./bashrc 
```
Verify your installation
```
$HADOOP -version
```
The output should be something like this
```
Hadoop 2.7.3
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r baa91f7c6bc9cb92be5982de4719c1c8af91ccff
Compiled by root on 2016-08-18T01:41Z
Compiled with protoc 2.5.0
From source with checksum 2e4ce5f957ea4db193bce3734ff29ff4
This command was run using /home/admins/hadoop-2.7.3/share/hadoop/common/hadoop-common-2.7.3.jar
```
## Step 5
**CONFIGURING HADOOP**   
You have to make the some changes to the followng files in **$HADOOP/etc/hadoop**   
### (1)core-site.xml
```
<property>
		<name>fs.defaultFS</name>
		<value>hdfs://192.168.1.103:8020</value>
	</property>
```
### (2)hdfs-site.xml
```
<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:/opt/hadoop/hadoop_store/hdfs/namenode</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/opt/hadoop/hadoop_store/hdfs/datanode</value>
	</property>
```
Make sure the directory is created
```
cd $HADOOP
mkdir -p /hadoop_store/hdfs/namenode 
chmod 755 /hadoop_store/hdfs/namenode
mkdir -p /hadoop_store/hdfs/datanode 
chmod 755 /hadoop_store/hdfs/datanode
```
### (3)mapred-site.xml
First make the file by copying mapred-site.xml.template to mapred-site.xml
```
cd $HADOOP
cp /etc/hadoop/mapred-site.xml.template /etc/hadoop/mapred-site.xml 
```
Then add the following lines in **mapred-site.xml**
```
<property>
	    <name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
```
### (4)yarn-site.xml
```
<property>
	  <name> yarn.resourcemanager.resource-tracker.address</name>
	  <value>192.168.1.103:8025</value>
	</property>

	<property>
	  <name> yarn.resourcemanager.scheduler.address</name>
	  <value>192.168.1.103:8030</value>
	</property>
	
	<property>
	  <name> yarn.resourcemanager.address</name>
	  <value>192.168.1.103:8050</value>
	</property>

	<property>
	  <name>yarn.nodemanager.aux-services</name>
	  <value>mapreduce_shuffle</value>
	</property>

	<property>
	  <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
	</property>

	<property>
	  <name>yarn.nodemanager.disk-health-checker.min-healthy-disks</name>
	  <value>0</value>
	</property>
```
### (5)slave
Add the ip adress of the slaves node in **/etc/hadoop/slave**
```
192.168.1.104
192.168.1.105
```
### (6)hadoop-env.sh
Set the JAVA_HOME variable path in the file.
```
export JAVA_HOME=/opt/jdk1.7.0_17
```
## Step 6
**INSTALLING HADOOP ON SLAVES**
First you have to make sure SSH server is installed on all the machines
```
sudo apt-get install openssh-server 
sudo systemctl restart ssh
```
Copy the hadoop file from master node to slaves, in the master node execute the following commands:
```
cd $HADOOP 
scp -r hadoop slave1:/opt/hadoop 
scp -r hadoop slave2:/opt/hadoop
```

## step 7
**FORMAT HADOOP FILE SYSTEM IN MASTER NODE**
```
hdfs namenode -format
```
---
## To start the HDFS and YARN deamons, run the followin in master node:
```
start-dfs.sh
start-yarn.sh
```
OR
```
start-all.dfs
```
---
### Every deamone has a port to run a GUI on: 
**namenode port:** 50070   
**resourcemanager:** 8088
### Disable the firewall if the GUI is not responding:	
```
sudo ufw disable
```
