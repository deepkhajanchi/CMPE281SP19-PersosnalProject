# Property of Partition Tolerance in CP and AP Database Systems.
## INDEX
### Timeline 1: 23-Feb-2019 to 10-Mar-2019
### Basic Concept
#### 1. NoSQL Database
##### - CAP Theorem
1. CA Property
2. CP Property
3. AP Property
### Timeline 2: 11-Mar-2019 to 30-Mar-2019
#### 2. Implementation 
##### 1. CP Database System - MongoDB
MongoDB Architecture
  1. Setup of MongoDB Cluster using AWS and Gitbash
  2. Insertion of Database
  3. Normal state
  4. Partitioning in Cluster
  5. Recovery of Partition
  6. Test Cases		
  7. Mongo DB Sharding
##### 2. AP Database System - Riak
Riak Architecture
  1. Setup of Riak Cluster using AWS and Gitbash
  2. VPC Peering (WoW Factor)
  3. Normal State
  4. Partioning in CLuster
  5. Recovery of Partition
  6. Test Cases
  
  ### Basic Concept
  #### 1. NoSQL Database
   A NoSQL database provides a mechanism for storage and retrieval of data that is modeled in other than the tabular relations used in relational databases. This approach has simplicity of design, horizontal scaling to clusters of machines (a problem for relational databases), and good control over availability. The data structures used by NoSQL databases (e.g. column, document, key-value or graph) are different from those used by default in relational databases, making some operations faster in NoSQL. The particular suitability of a given NoSQL database depends on the problem it must solve. Sometimes the data structures used by NoSQL databases are also seen as more flexible than relational database.

Properties > schema-less, self-contained data, eventually consistent, highly scalable, cost effective, replication of data, basically available, soft state, open source, polyglot persistent.

Polyglot Persistence > This is a term used for multiple data storage technologies when storing different kind of data. It is chosen based on the way data is being used by applications or components of an application.  Instead of trying to store all this data in one database, which would require a lot of data conversion to make the format of the data all the same, store the data in the database best suited for that type of data.
 
 ##### - CAP Theorem
  
  Basic idea of CAP theorem is that a distributed database system can only have 2 of 3: Consistency, Availability and Partition Tolerance. This theorem is very important in Big Data terminologies, especially when we need to make trade offs between the three, based on unique use case.
  
Consistency --> Each client always has the same view of the data.

Availibility --> Every non-failing node returns a response for all read and write requests in a reasonable amount of time.

Partition Tolerance --> Tolerance to a network Partition

1. CA property: Consistent and Available

2. CP property: Consistent and Partition Tolerant

3. AP property: Available and PArtition Tolerant

#### 2. Implementation
##### 1. CP Database System - MongoDB

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/MongoDB%20Arch/MongoDB%20Architecture.png)

###### (1) Setup of MongoDB Cluster using AWS and Gitbash
###### References
    https://docs.mongodb.com/manual/tutorial/deploy-replica-set-with-keyfile-access-control/#deploy-repl-set-with-auth
    https://gist.github.com/calvinh8/c99e198ce5df3d8b1f1e42c1b984d7a4    
    https://eladnava.com/deploy-a-highly-available-mongodb-replica-set-on-aws/
    http://www.serverlab.ca/tutorials/linux/database-servers/how-to-create-mongodb-replication-clusters/
    https://docs.mongodb.com/manual/reference/default-mongodb-port/

###### Launch Ubuntu Server 16.04 LTS
  
    1. AMI:             Ubuntu Server 16.04 LTS (HVM) 
    2. Instance Type:   t2.micro
    3. VPC:             CMPE281 (Region: N. Virginia)
    4. Network:         public subnet
    5. Auto Public IP:  no
    6. Security Group:  mongodb-cluster 
    7. SG Open Ports:   22, 27017
    8. Key Pair:        cmpe281-us-west-1
    
Security Group rules port 22 and port 27017 are added. Port 22 is TCP used for SSH and port 27017 is also TCP which is default MongoDB port.

Allocate Elastic IP for Mongo instance

    'Allocate Elastic IP: Scope VPC'
    '18.214.168.141 mongo_1'

SSH into Ubuntu

    Set Path of Keyfile from local system
        cd '/c/Users/Dell/Desktop'
	cd 'SJSU sem1'/CMPE281/
	chmod 400 cmpe281-us-east-1.pem     //changing mode for keypair
        
        ssh -i "cmpe281-us-east-1.pem" ubuntu@ec2-18-214-168-141.compute-1.amazonaws.com    (SSH into ubuntu server through public DNS)  // for node1
        ssh -i "cmpe281-us-east-1.pem" ubuntu@ec2-34-204-149-237.compute-1.amazonaws.com    // for node2
        ssh -i "cmpe281-us-east-1.pem" ubuntu@ec2-54-84-78-225.compute-1.amazonaws.com      // for node3
        ssh -i "cmpe281-us-east-1.pem" ubuntu@ec2-34-238-211-79.compute-1.amazonaws.com     // for node4
        ssh -i "cmpe281-us-east-1.pem" ubuntu@ec2-34-225-245-12.compute-1.amazonaws.com     // for node5
        
![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(1)AWS%20instance/mongo_aws1.png)

Install MongoDB 

    === 3.4 ===
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6

    echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

    sudo apt-get update
    sudo apt-get install -y mongodb-org

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4

    echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb.list

    sudo apt update
    sudo apt install mongodb-org

    === Install a Specific Version ===

    sudo apt install mongodb-org=4.0.1 mongodb-org-server=4.0.1 mongodb-org-shell=4.0.1 mongodb-org-mongos=4.0.1 mongodb-org-tools=4.0.1

    === Start/Stop and Install Verification ===

    sudo systemctl enable mongod
    sudo systemctl start mongod 
    sudo systemctl stop mongod
    sudo systemctl restart mongod

    mongod --version

MongoDB Keyfile

    openssl rand -base64 741 > keyFile
    sudo mkdir -p /opt/mongodb
    sudo cp keyFile /opt/mongodb
    sudo chown mongodb:mongodb /opt/mongodb/keyFile
    sudo chmod 0600 /opt/mongodb/keyFile

Configuration of mongod.conf
    `sudo vi /etc/mongod.conf`

    1. remove or comment out bindIp: 127.0.0.1
    replace with bindIp: 0.0.0.0 (binds on all ips) 

    # network interfaces
    net:
        port: 27017
        bindIp: 0.0.0.0

    2. Uncomment security section & add key file

    security:
        keyFile: /opt/mongodb/keyFile

    3. Uncomment Replication section. Name Replica Set = cmpe281

    replication:
        replSetName: cmpe281

    4. Create mongod.service

    sudo vi /etc/systemd/system/mongod.service

      [Unit]
          Description=High-performance, schema-free document-oriented database
          After=network.target

      [Service]
          User=mongodb
          ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

      [Install]
          WantedBy=multi-user.target

    5. Enable Mongo Service
 
        sudo systemctl enable mongod.service

    6. Restart MongoDB to apply our changes
 
        sudo service mongod restart
        sudo service mongod status

Save to AMI Image

    AMI: mongo-ami 

Initialize the replica set. Using mongo-ami, launch another four free tier instances and allocate and assign Elastic IPs.
    Allocate Elastic IP: Scope VPC

    34.04.149.237 mongo_2
    54.84.78.225 mongo_3
    34.238.211.79 mongo_4
    34.225.245.12 mongo_5

    Edit /etc/hosts in each EC2 Instance adding local host names for Public IPs in each node including primary

        18.214.168.141 primary    
        34.04.149.237 secondary1
        54.84.78.225 secondary2
        34.238.211.79 secondary3
        34.225.245.12 secondary4

    Test each instance using the following to make sure hostnames are correct:

        hostname -f    

    If not, change the hostnames on each instance as follows:
 
        sudo hostnamectl set-hostname <new hostname>
        sudo hostname -f
        reboot (to make sure the change sticks)

    Initialize the Replica Set

        mongo  (run as local client on primary)

        rs.initiate( {
           _id : "cmpe281",
           members: [
              { _id: 0, host: "primary:27017" },
              { _id: 1, host: "secondary1:27017" },
              { _id: 2, host: "secondary2:27017" },
              { _id: 3, host: "secondary3:27017" },
	      { _id: 4, host: "secondary4:27017" },
           ]
        })

        rs.status()

    Troubleshooting:  If you have connectivity issues, check that mongo is up and running
    and use "telnet" to try to connect to sceondaries from primary (and vice versa)

        sudo service mongod restart
        sudo service mongod status

        telnet <host or ip> 27017

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(1)AWS%20instance/mongo_aws2.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(1)AWS%20instance/mongo_aws3.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(1)AWS%20instance/mongo_aws4.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(1)AWS%20instance/mongo_aws5.png)

Admin Account Creation

    The default MongoDB configuration is wide open, meaning anyone can access 
    the stored databases unless your network has firewall rules in place.

    Create an admin user to access the database.

    mongo

    Select admin database.

    use admin

    Create admin account.

        db.createUser( {
            user: "admin",
            pwd: "*****",
            roles: [{ role: "root", db: "admin" }]
        });

    Login to Primary as Admin:
        mongo -u <user> -p <password> --authenticationDatabase admin

    Login to Mongo Remote ( This is only for remote ) 
        mongo -u <user> -p <password> <mongo host ip> --authenticationDatabase admin

    Used to login into Primary and secondary nodes:
        mongo -u admin -p password --authenticationDatabase admin

    Change Password (if needed)
 
        use admin
        db.changeUserPassword( "admin", "*****" )    

Connection to Primary and Test DB Commands

    db.test.save( { a : 1 } )   				  // save simple document
    db.test.find()              				  // find document
    db.test.replaceOne( { a : 1 }, { a : 2 } )			  // update document

##### (2)Insertion of Data in MongoDB:
After going through all these previous processes now let us log in into mongo account using username and password in all of the nodes including primary and secondary.

    mongo -u admin -p password --authenticationDatabase admin
    use admin   //Record insertion in admin database

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(2)Login%20to%20nodes/mongo_login.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(2)Login%20to%20nodes/mongo_login1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(2)Login%20to%20nodes/mongo_login2.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(2)Login%20to%20nodes/mongo_login3.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(2)Login%20to%20nodes/mongo_login4.png)

Data insertion from
    `bios.js`
    
    Data is inserted only in the primary node. Replication of data will be reflected among all the secondary nodes regarding different states i.e normal state, secondary node partitioning and     primary node partitioning .

Allowing Queries from Replica nodes
    
    db.test.find() ;
	
	Error: error: {
		"operationTime" : Timestamp(1537972200, 1),
		"ok" : 0,
		"errmsg" : "not master and slaveOk=false",
		"code" : 13435,
		"codeName" : "NotMasterNoSlaveOk"
	}

    Set before querying in secondary nodes:
        rs.slaveOk()
    
##### (3)Normal State

In the normal state, I have simply used admin database logging in admin account. Then inserted record from file bios.js. and ran following query:
        `db.bios.find({},{"name.first":1}).sort({"name.first":1})`

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(3)Normal%20State/normalstate_query1.png)

For secondary nodes,

    rs.slaveOk()
    db.bios.find({},{"name.first":1}).sort({"name.first":1})

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(3)Normal%20State/normalstate_query2.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(3)Normal%20State/normalstate_query3.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(3)Normal%20State/normalstate_query4.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(3)Normal%20State/normalstate_query5.png)

As per screenshots, query insertion has been done in all nodes. Hence fulfilment of normal state.

##### (4)Partition of node secondary4 from the MongoDB cluster 

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/MongoDB%20Arch/Node%20Partition.png)

To make a partition, I have run iptables command after looging out from mongo in node secondary4. After that data insertion or updation will not be replicated in node secondary4. Remaining cluster will be remained as connected as per normal state.

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(4)Disconnection%20of%20secondary4/(1)primary_changerefleted.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(4)Disconnection%20of%20secondary4/(2)secondary1_changerefleted.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(4)Disconnection%20of%20secondary4/(3)secondary2_changereflected.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(4)Disconnection%20of%20secondary4/(4)secondary3_changerefleted.png)

    sudo iptables -A INPUT -s primary -j DROP
    sudo iptables -A INPUT -s secondary1 -j DROP
    sudo iptables -A INPUT -s secondary2 -j DROP
    sudo iptables -A INPUT -s secondary3 -j DROP
    
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(4)Disconnection%20of%20secondary4/(5)secondary4_notrefleted.png)   

To check status
    `rs.status()`

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(4)Disconnection%20of%20secondary4/(6)node4_unhealthy.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(4)Disconnection%20of%20secondary4/(7)node4_disconnected.png)

It shows state as "Not reachable/healthy". New inserted record with "name":"Deep" has not been inserted due to partition.

##### (5) Partition Recovery

To make connection with node secondary4, command of iptable needs to be run on that node itself.

    sudo iptables -D INPUT -s primary -j DROP
    sudo iptables -D INPUT -s secondary1 -j DROP
    sudo iptables -D INPUT -s secondary2 -j DROP
    sudo iptables -D INPUT -s secondary3 -j DROP

This will remove partition and after using database if we try to do replication of data by insert query command, will start reading record from primary node as per attached screenshot.

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(5)Secondary4--%20Partition%20recovery/secondary4_partition_to_normalstate.png)
    
##### Partition of primary node from cluster

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/MongoDB%20Arch/Primary%20Partition.png)

This functionality is interesting in MongoDB. If we try to seperate primary node from the cluster than neither it will disconneted nor will stop replication. But it will make all other secondary node to do election among them to become a primary. In between this process primary node changes itself to a secondary node.

    sudo iptables -A INPUT -s secondary1 -j DROP
    sudo iptables -A INPUT -s secondary2 -j DROP
    sudo iptables -A INPUT -s secondary3 -j DROP
    sudo iptables -A INPUT -s secondary4 -j DROP

This shifting of primary node to one of the secondary node is made detailed through following screenshot.

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(6)Disconnection%20of%20primary/primary_becomes_secondary.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(6)Disconnection%20of%20primary/one_of_the_secondary_primary.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(6)Disconnection%20of%20primary/secondary_remains_secondary.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(6)Disconnection%20of%20primary/secondary_remains_secondary_.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Mongo%20Database/(6)Disconnection%20of%20primary/secondary_remains_secondary__.png)

Now let us get into the test cases:

##### (6) Testcases

- How does the system function during normal mode (i.e. no partition)?

During normal mode, written data in primary node is replicated in all other secondary node as MongoDB follows Master-Slave Architecture. Any of the nodes is able to read data from primary node. After checking '''rs.status()''' from every node. health status is '''1''' and it shows '''connected''' in every node as well.

- What happens to the master node during a partition? 

Master node becomes slave node as soon as command is fired and election process will be held among other slave to take place of master. Doing so primary node will not get apart from cluster but it will change itself to secondary node.

create partition from primary node

    sudo iptables -A INPUT -s secondary1 -j DROP
    sudo iptables -A INPUT -s secondary2 -j DROP
    sudo iptables -A INPUT -s secondary3 -j DROP
    sudo iptables -A INPUT -s secondary4 -j DROP

- Can stale data be read from a slave node during a partition?

Yes executing command
`rs.slaveOk()`

It allows us to read data from the secondary node written in primary node.
In some of the cases when the written data or updated information on primary node may not be reflected on secondary node.And this is due to replica lag.

Replica lag:
the last written operation applied on the particular secondary node which was hidden by the most recent operation applied on the primary. The procedure of reading from a lagging secondary node is 'Eventual Consistency". It will be consistent under normal circumstances. 

- What happens to the system during partition recovery?

sudo iptables -D INPUT -s primary -j DROP
sudo iptables -D INPUT -s secondary1 -j DROP
sudo iptables -D INPUT -s secondary2 -j DROP
sudo iptables -D INPUT -s secondary3 -j DROP

With the help of `iptables -D` secondary4 starts reading data from written data in primary. Cluster starts working as a normal state.
During the recovery, the primary node will be able to control the secondary nodes but there are the chances that all the secondary nodes might not be get into replication of primary node. To obtain a consistent data, all the secondary nodes must be syncing to the primary node.  

##### (7) MongoDB Sharding

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/MongoDB%20Shard/MongoDB%20sharding%20architecture.png)

I have deployed a Mongos Query Router and a cluster of 2 configuration servers to store the metadata. 
Two data shard clusters are created. Each cluster has two instances, one primary and one secondary for data availability and replication.

Mongos and mongod  on port 27017

Configuration servers are running on port 27019

Data shard clusters are running on port 27018

1. Create a Security Group

In default security group add SSH(22) to public

	SG: 		sharding
	OPEN PORTS:	27017-27019
	Edit internal bound traffic to the group id of this SG so that no external traffic is allowed.
 
2. MongoDB AMI

a. Launch Server (This will be Configuration server 1)

	* AMI:             Amazon Linux AMI 2018.03.0 (HVM), SSD Volume Type
	* Instance Type:   t2.micro
	* VPC:             default
	* Network:         public subnet
	* Auto Public IP:  Auto assign ip
	* Security Group:  default+mongodb-internal access
	* Key Pair:        cmpe281-us-west-2 
	
b. SSH into Mongo Instance using command line OR bash

	ssh -i <key>.pem ec2-user@<public ip>
	
c. Install MongoDB

	Ref Link: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-amazon/
	
      Add the following in the file location so that we can use yum to install mongo packages

	sudo nano /etc/yum.repos.d/mongodb-org-4.0.repo
	
	[mongodb-org-4.0]
	name=MongoDB Repository
	baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/4.0/x86_64/
	gpgcheck=1
	enabled=1
	gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
	 
Install mogodb on our server

	sudo yum -y update && sudo yum install -y mongodb-org 
	
d. Create an image of this instance

	AMI: mongodb-ami
	
3. Create 6 instances using mongodb-ami and SG: default + internal

	config-server-2 
	
	shard-server-1.1 & shard-server-1.2
	
	shard-server-2.1 & shard-server-2.2
	
	mongos 
	
4. Configure the config-servers 1 and 2

###### PART 1

a. Check if any process is active:

If mongod is running. stop it using the command: sudo service mongod stop

	sudo lsof -iTCP -sTCP:LISTEN | grep mongo

b. Create a directory which contains the metadata and change its ownership.

	sudo mkdir -p /data/db
	sudo chown -R mongod:mongod /data/db

c. Specify mongod settings in the mongod.conf file

	sudo nano /etc/mongod.conf 

	**********************************
	dbPath: /data/db

	port:27019
	bindIp: 0.0.0.0

	replication:
		replSetName: crs
	sharding:
		clusterRole: configsvr
	**********************************

d. Start mongod

	sudo service mongod start
	sudo service mongod status

Do the same with config-server-2 

###### PART 2

a. In any one of the config server connect to mongo shell by specifying port number

	mongo -port 27019

b. Initiate the replicaset using rs.initiate //use the private ip addresses of config servers 1 and 2

	rs.initiate(
		{
			_id: "crs",
			configsvr: true,
			members: [
				{ _id:0, host : "172.31.29.164:27019"},
				{ _id:1, host : "172.31.28.36:27019"}
			]
		}
	)

c. Check with rs.status()

5. #### Configure the sharded data replica sets

##### PART 1

a. Check if any process is active:(should not be). If mongod is running, stop it using the command: sudo service mongod stop

	sudo lsof -iTCP -sTCP:LISTEN | grep mongo

b. Create a directory which contains the metadata and change its ownership.

	sudo mkdir -p /data/db
	sudo chown -R mongod:mongod /data/db

c. Specify mongod settings in the mongod.conf file

	sudo nano /etc/mongod.conf 

	**********************************
	dbPath: /data/db

	port:27018
	bindIp: 0.0.0.0

	replication:
		replSetName: rs0/rs0(for shard 1) rs1/rs1(for shard 2) 
	sharding:
		clusterRole: shardsvr
	**********************************
	
d. Start mongod

	sudo service mongod start
	sudo service mongod status

Do the same for all shard instances.

###### PART 2

a. In any one of the config server connect to mongo shell by specifying port number

	mongo -port 27018

b. Deploy the replicaset using rs.initiate //use the private ip addresses of shard instances

 
	/shard 1/
	rs.initiate(
		{
			_id: "rs0", 
			members: [
				{ _id:0, host : "34.234.93.162:27018"},
				{ _id:1, host : "54.242.243.138:27018"}
			]
		}
	)

	/shard 2/
	rs.initiate(
		{
			_id: "rs1", 
			members: [
				{ _id:0, host : "3.84.121.202:27018"},
				{ _id:1, host : "54.90.153.254:27018"}
			]
		}
	)

c. Check with rs.status()

6.  Configure mongos query router and connect it to sharded cluster

##### PART 1

a. check if any process is active:(should not be). 

If mongod is running, stop it using the command: `sudo service mongod stop`

	sudo lsof -iTCP -sTCP:LISTEN | grep mongo

b. Specify mongod settings in the mongod.conf file

	sudo nano /etc/mongod.conf 
	
	**********************************
	#storage
	#dbPath:
	#journal
	#enabled

	port:27017
	bindIp: 0.0.0.0

	sharding:
		configDB: crs/54.147.40.77:27019,34.207.210.14:27019   //config servers ip addr
	**********************************

c. Start mongos service
	
	sudo mongos --config /etc/mongod.conf --fork --logpath /var/log/mongodb/mongod.log
 
###### PART 2

a. Connect mongo shell to mongos

	mongo -port 27017

b. Add each shard to the cluster. Use the addShard function

	sh.addShard("rs0/34.234.93.162:27018,54.242.243.138:27018");//private ip of shard1
	sh.addShard("rs1/3.84.121.202:27018,54.90.153.254:27018"); //private ip of shard2
 
c. View shard list

	db.adminCommand({ listShards:1})

###### Test MongoDB Data Sharding

1 Enable sharding on database level.

2 Create database

	use database1

3 Enable sharding for this db

	sh.enableSharding("database1")

4 Select a shard key. Here we are taking the id as our shard key for our bios collection.

	sh.shardCollection("database1.bios", {_id : "hashed" } )
	
5 Insert data into the bios collection.

6 To see the data sharding, SSH into shard instance one and two and find the data.

	use database1
	db.bios.find()
	
SSH into secondary instances of the data shard cluster and check if the sharded data is replicated.

	use database1
	rs.slaveOk()
	db.bios.find()

The slaveOk() command indicates that “eventually consistent” read operations are acceptable for the current application.
	
**Observation**

We observe that some data is in shard cluster 1 and other data is in the other data shard.
	
###### Challenges

1. When configuring the mongod.conf file, mongod was already running. I had to stop mongod before making the changes.

2. The port for config server was set to 27017 instead of 27019. So I could not connect to mongo through that port.

3. After initiating the replica set for the shards, both the instances were secondary on rs.status(). When I ran the same command again, primary node was visible. Time for initiate.

#### 2. AP Database System - Riak 

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/Riak%20Arch/Riak%20Architecture%20(Normal).png)

##### (1) Setup of Riak Cluster using AWS and Gitbash

I have went through these links to create Riak cluster.

    http://basho.com/posts/technical/riak-on-aws-deployment-options/
    http://docs.basho.com/riak/kv/2.2.3/developing/usage/
    http://docs.basho.com/riak/kv/2.2.3/setup/installing/amazon-web-services/
    http://docs.basho.com/riak/kv/2.2.3/using/running-a-cluster/#configure-the-first-node
    http://docs.basho.com/riak/kv/2.2.3/using/cluster-operations/adding-removing-nodes/
    http://docs.basho.com/riak/kv/2.2.3/developing/usage/conflict-resolution/
    https://aws.amazon.com/marketplace/pp/B00YFZ60X2?ref=cns_srchrow
    http://docs.basho.com/riak/kv/2.2.3/setup/installing/amazon-web-services/

Launch Riak Marketplace AMI (5 nodes)
    
I have made peer connection between two VPCs and created a riak cluster within those VPCs.
Region-1 is N.Virginia and Region-2 is Ohio. In the first region there are 3 Riak nodes and in the second node there are 2 Riak nodes.

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/1_Riak%20Cluster/Riak_3nodes.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/2_AWS%20instances/(1)Riak1-Sub1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/2_AWS%20instances/(2)Riak2-Sub1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/2_AWS%20instances/(3)Riak3-Sub1.png)

Let us go through first region (3 nodes)
    
    1. AMI:             Riak KV 2.2 Series
    2. Instance Type:   t2.micro
    3. VPC:             CMPE281
    4. Network:         public subnet
    5. Auto Public IP:  no
    6. Security Group:  Riak KV1 
    7. SG Open Ports:   0-65535 (see below)
    8. Key Pair:        cmpe281-us-west-1

    Riak Cluster Security Group (Open Ports):

    22 (SSH)
    8087 (Riak Protocol Buffers Interface)
    8098 (Riak HTTP Interface)

    You will need to add additional rules within this security group to allow your Riak instances to communicate. For each port range below, create a new Custom TCP rule with the source set     to the current security group ID (found on the Details tab).

    Port range: 4369
    Port range: 6000-7999
    Port range: 8099

    http://docs.basho.com/riak/kv/2.2.3/setup/installing/amazon-web-services/
    http://docs.basho.com/dataplatform/1.0.0/configuring/default-ports/

    NOTE:  Port Ranges above, as documented, will not work for Creating a Cluster.
    There seems to be missing port(s) that needs to be open.  As such, just add
    the following Rule instead of the 4369, 6000-7999 and 8099 rules.

    Port range: 0-65535        

SSH into Riak instance

    Set Path of Keyfile from local system
        cd '/c/Users/Dell/Desktop'
  	cd 'SJSU sem1'/CMPE281/
	  chmod 400 cmpe281-us-east-1.pem     //changing mode for keypair

Allocate and assign elastic IP to every nodes

    '52.23.47.105 Riak1-Sub1'
    '3.95.75.180 Riak2-Sub1'
    '54.161.8.128 Riak3-Sub1'    

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/1_Riak%20Cluster/Riak_2nodes.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/2_AWS%20instances/(4)Riak4-Sub2.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/2_AWS%20instances/(5)Riak5-Sub2.png)

For the second region (2 nodes)   
        
    1. AMI:             Riak KV 2.2 Series
    2. Instance Type:   t2.micro
    3. VPC:             CMPE281-1
    4. Network:         public subnet
    5. Auto Public IP:  no
    6. Security Group:  Riak KV2
    7. SG Open Ports:   0-65535 (see below)
    8. Key Pair:        cmpe281-1-us-west-2pem

    Riak Cluster Security Group (Open Ports):

    22 (SSH)
    8087 (Riak Protocol Buffers Interface)
    8098 (Riak HTTP Interface)

    You will need to add additional rules within this security group to allow your Riak instances to communicate. For each port range below, create a new Custom TCP rule with the source set       to the current security group ID (found on the Details tab).

    Port range: 4369
    Port range: 6000-7999
    Port range: 8099

    http://docs.basho.com/riak/kv/2.2.3/setup/installing/amazon-web-services/
    http://docs.basho.com/dataplatform/1.0.0/configuring/default-ports/

    NOTE:  Port Ranges above, as documented, will not work for Creating a Cluster.
    There seems to be missing port(s) that needs to be open.  As such, just add
    the following Rule instead of the 4369, 6000-7999 and 8099 rules.

    Port range: 0-65535

SSH into Riak instance

    Set Path of Keyfile from local system
        cd '/c/Users/Dell/Desktop'
	cd 'SJSU sem1'/CMPE281/
	chmod 400 cmpe281-1-us-east-1pem.pem     //changing mode for keypair

Allocate and assign elastic IP to every nodes

    '3.18.218.150 Riak4-Sub2'
    '3.19.9.149 Riak5-Sub2'

#####  (2) VPC Peering (WoW Factor)

      https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html
      
To Enable peering between two VPCs, there are some modification under VPC tab.

Firstly, VPC from one region (Region-1 N. Virginia) send request to VPC in another region (Region-2 Ohio). Before that I have seetled CIDR block IPv4, Public Subnet IPv4 and Private Subnet IPv4.

    IPv4 CIDR block: 10.0.0.0/16
    Public Subnet: 10.0.0.0/24
    Private Subnet: 10.0.1.0/24
  
Same changes I have made in another region's VPC.

    IPv4 CIDR block: 10.1.0.0/16
    Public Subnet: 10.1.0.0/24
    Private Subnet: 10.1.1.0/24
  
It is clear that IPv4 for the both region are different from each other. Only after that VPC perring can be performed.
VPC accepter ID needs to be noted in first VPC to make successfull peering. Screenshot has been attached for detailed information.


![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/1VPC-subnet-peer1.png)


![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/2VPC-subnet-peer2.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/3Peering%20request%20from%20N_Virginia.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/4Peering%20request.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/5Requester%20Region.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/6Accepter%20Region.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/7RouteTable_region1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/3_VPC%20peering/8RouteTable_region2.png)


Thus, there is a successful VPC peering. Now, for data replication among every from both the regions, I modifies routes under Route Table tab. CIDR value of second region `10.1.0.0/16` is added into route table of first region and same modification with second region's route table.

I have added `10.0.0.0/16` into route table of second region.

Thus, by doing these process, successful cluster has been settled up among all the nodes from both the regions. Now let us go through various states for Riak cluster.

Then after there are some changes have made in `riak/riak.conf` file in each and every Riak node to enable listener and Riak ports.

Commit plan for node1:

    listener.http.internal = 10.0.0.240:8098
    listener.protobuf.internal = 10.0.0.240:8087
    riak.conf:nodename = riak1@10.0.0.240
    
    sudo riak start
    sudo riak ping
    sudo riak-admin status
    
    sudo riak-admin cluster plan
    sudo riak-admin cluster status
    
 Commit plan for node2:
  
    listener.http.internal = 10.0.0.173:8098
    listener.protobuf.internal = 10.0.0.173:8087
    riak.conf:nodename = riak1@10.0.0.173
    
    sudo riak start
    sudo riak ping
    sudo riak-admin status
    
    sudo riak-admin cluster join riak@10.0.0.240  (node 2)
    sudo riak-admin cluster plan
    sudo riak-admin cluster status    
    
 Commit plan for node3
 
    listener.http.internal = 10.0.0.41:8098
    listener.protobuf.internal = 10.0.0.41:8087
    riak.conf:nodename = riak1@10.0.0.41
    
    sudo riak start
    sudo riak ping
    sudo riak-admin status
    
    sudo riak-admin cluster join riak@10.0.0.240  (node 3)
    sudo riak-admin cluster plan
    sudo riak-admin cluster status
     
 Commit plan for node4    
 
    listener.http.internal = 10.1.0.27:8098
    listener.protobuf.internal = 10.1.0.27:8087
    riak.conf:nodename = riak1@10.1.0.27
    
    sudo riak start
    sudo riak ping
    sudo riak-admin status
    
    sudo riak-admin cluster join riak@10.1.0.240  (node 4)
    sudo riak-admin cluster plan
    sudo riak-admin cluster status
    
 Commit plan for node5
 
    listener.http.internal = 10.1.0.120:8098
    listener.protobuf.internal = 10.1.0.120:8087
    riak.conf:nodename = riak1@10.1.0.120
    
    sudo riak start
    sudo riak ping
    sudo riak-admin status
    
    sudo riak-admin cluster join riak@10.1.0.240  (node 5)
    sudo riak-admin cluster plan
    sudo riak-admin cluster status
 
 Now we have a Raik cluster with 5 nodes from 2 virtual private clouds. All the ther nodes are joined with node `10.0.0.240`.
 Commit plan for Riak CLuster. Status of Cluster can bechecked through:
 
    sudo riak-admin cluster plan
    sudo riak-admin cluster status
     If this looks good:
    sudo riak-admin cluster commit
     To check the status of clustering use:
    sudo riak-admin member_status
    
 Riak Usage:
 
    curl -i http://riak1@10.0.0.240:8098/ping
    curl -i http://riak1@10.0.0.240:8098/buckets?buckets=true 
     
  ##### (3) Normal State
  In the Riak, we can write and read record from any of the node. Riak does not follow master-slave architecture. All the nodes are at same level.
  
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(3)Cluster%20Status/(1)cluster_status_ping.png)
  
  So we will write data in the first node and that will be read from all other nodes.
  
      curl -v -XPUT -d '{"name":"Heath"}' \http://riak1@10.0.0.240:8098/buckets/bucket/keys/key1?returnbody=true
      
![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(4)insertion%20Record/(1)Query%20in%20node1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(4)insertion%20Record/(2)Query%20reflection%20in%20node2.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(4)insertion%20Record/(3)Query%20reflection%20in%20node3.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(4)insertion%20Record/(4)Query%20reflection%20in%20node4.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(4)insertion%20Record/(5)Query%20reflection%20in%20node5.png)
 
 To read data from all other nodes:
     
    curl -i http://riak2@10.0.0.173:8098/buckets/bucket/keys/key1
    curl -i http://riak3@10.0.0.41:8098/buckets/bucket/keys/key1
    curl -i http://riak4@10.1.0.27:8098/buckets/bucket/keys/key1 
    curl -i http://riak5@10.1.0.120:8098/buckets/bucket/keys/key1
    
 You can see in the screenshots, Riak is working properly under normal state.
 
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(2)Riak%20Start/(1)First%20node_running.png)
 
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(2)Riak%20Start/(2)Nodes_joined1.png)
 
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(2)Riak%20Start/(3)Nodes_joined2.png)
 
 To double check, let us insert record from another node and it will be reflected in all other nodes.
 Insertion from `10.0.0.173`
 
     curl -v -XPUT -d '{"name":"Lara"}' \http://riak2@10.0.0.173:8098/buckets/bucket/keys/key1?returnbody=true
 
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(5)Record%20insertion%20from%20other%20node/(1)insert%20and%20reflected.png)
  
     
 Read from all other nodes.
 
    curl -i http://riak1@10.0.0.240:8098/buckets/bucket/keys/key1
    curl -i http://riak3@10.0.0.41:8098/buckets/bucket/keys/key1
    curl -i http://riak4@10.1.0.27:8098/buckets/bucket/keys/key1 
    curl -i http://riak5@10.1.0.120:8098/buckets/bucket/keys/key1
    
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(5)Record%20insertion%20from%20other%20node/(2)replication%20to%20node1.png)
 
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(5)Record%20insertion%20from%20other%20node/(3)replication%20to%20node3.png)
 
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(5)Record%20insertion%20from%20other%20node/(4)replication%20to%20node4.png)
 
 ![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/4_Normal%20State/(5)Record%20insertion%20from%20other%20node/(5)replication%20to%20node5.png)

Hence, Normal State test case has been fulfilled.

##### (4) Partioning in Cluster

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/Riak%20Arch/Riak%20Architecture%20(Partition).png)

Now let us make partition between two VPCs to check partition tolerance of Riak Database system.
To do so I will remove CIDR block from routes.

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(1)Deletion%20in%20Route%20table.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(10)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(11)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(2)Effect%20in%20node%20in%20First%20VPC.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(5)Effect%20in%20node%20in%20Second%20VPC.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(7)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(8)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/5_Partitioning-queries%20from%20region1/(9)cluster-status.png)

As per screenshot I have removed CIDR blocks from cross region which I have added earlier.
By this there will not be any data replication in cross region.

After record insertion in a node from one region, data only will replicated in the same region. 

If we try to read data from another region following record will be shown.

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/6_Partitioning-queries%20from%20region2/(1)Effect%20in%20region2%20node.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/6_Partitioning-queries%20from%20region2/(2)Effect%20in%20region2%20node.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/6_Partitioning-queries%20from%20region2/(3)Effect%20in%20region1%20node.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/6_Partitioning-queries%20from%20region2/(4)Effect%20in%20region1%20node.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/6_Partitioning-queries%20from%20region2/(5)Effect%20in%20region1%20node.png)

We can check status of cluster:

    sudo riak-admin cluster status
Thus, we have check Partition status in Riak cluster.

##### (5) Recovery of Partition
To recover partition, I have added CIDR IPv4 block of Region-2 (Ohio) in the routes of Region-1(N. Virginia). By doing so I have recovered cluster partition among all the nodes.

I have added record in a region and now it is replicated among all the nodes and `rs.status()` shows connected status.
  
Therefore all the conditions have been fulfilled. Now let us go through Test cases.  

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(1)Add%20CIDR%20rule%20in%20region1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(2)Data%20from%20node%20in%20region2.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(3)Data%20read%20in%20region2%20node.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(4)Data%20from%20node%20in%20region1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(5)Data%20from%20node%20in%20region1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(6)Data%20from%20node%20in%20region1.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(7)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(8)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(9)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(10)cluster-status.png)

![ss](https://github.com/nguyensjsu/cmpe281-deepkhajanchi/blob/master/Personal_Project/Riak%20Database/7_Partition%20recovery/(11)cluster-status.png)


##### (6) Test Cases
 
- How does cluster fucntion during normal mode?

Data gets replicated across all the nodes. If a data has been overwritten by another node, it gets reflected in all other node because riak is available rather than condsistance. As the all nodes in the replication set are on the same level, write can be happened from any of the node.

- What happens to the nodes during a partition? 

We have created partition using route table, when we remove rule from that two region gets separated and record can be replicated among one region node regarding written record node.
if we check status from region 1 then nodes from region2 are down
and if we check status from region 2 then nodes from region1 are down.

- Can stale data be read from a node during a partition?

Yes, Stale data can be read from the nodes. we were able to read the data from both the regional nodes.
by commnad,
   `sudo riak start`
   `sudo riak stop`

- What happens to the system during partition recovery?

By reverting route table rule to its original rule, all the nodes fron both region are now working as a cluster and cluster status is up!
