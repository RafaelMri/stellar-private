# Introduction
This document explains the installation of [Stellar](https://www.stellar.org) private network. Such a private network maybe used for testing purposes or launched to introduce a new currency. The goal of this article is to have a running Stellar network with Ubuntu-16.04 LTS Virtual Machines on AWS. Most of the instructions could be replicated for an on-premise installation or even other public cloud providers; however, that is out of scope of this article. The installation of Stellar Core will be categorised into two broad categories, viz. instance hosting the Stellar Core and the database instance that is used by the installation of Stellar Core. After installation, the configuration files are updated to reflect the new private network.

**NOTE** that, these steps are not meant for an industrial grade installation.

# Installation, set-up and running the network
At a high level, the steps for launching a private network are as follows:

* Stellar database installation
* Stellar core installation
* Stellar configuration file set-up
* Stellar start-up

With the installation in place, the configuration file is updated for all instances.

## Stellar database installation
Since the Stellar Core installation requires a database instance to communicate with, the first step is to have a running database instance. This database instance can be installed locally on the instance hosting Stellar Core or could be remote. At least, one database instance should be installed remotely so that a Horizon instance could connect to it.

For more information on installation of Horizon, refer ()

### Local database
It is easier and cheaper to run a `SQLite` database locally. It is also possible to run a `PostgreSQL` database locally; however, in this document, we will be referring to `SQLite` database installed locally.

* To install `SQLite` locally, run the following commands:
    ```bash 
    sudo apt-get update
    sudo apt-get install sqlite3 libsqlite3-dev
    ```

* Create a folder to host the database: `mkdir /home/ubuntu/stellardb`

For more information on installing `SQLite3` on Ubuntu-16.04 LTS VM, refer this link:(https://www.digitalocean.com/community/tutorials/how-and-when-to-use-sqlite)

### Remote database
To install a remote database for Stellar, the easiest option is to spin up an AWS RDS instance with `PostgreSQL` as the database engine. For details on how to spin up such an instance, refer this link:(https://aws.amazon.com/getting-started/tutorials/create-connect-postgresql-db/)

During installation, note down the following information of the AWS RDS installation:

 		End-point: <db-instance>.<aws-identifier>.<aws-region-name>.rds.amazonaws.com
 		Port: 5432
 		DB instance identifier: <db-instance_id>
 		Master user name: <db_master_user_name>
 		Master password: <db_master_user_password>
 		DB name: <db-name>
          
## Stellar Core installation
With a database installaed, we are ready to install the Stellar Core software. The installation steps are as follows. For more information on installation, refer (https://www.stellar.org/developers/stellar-core/software/admin.html).

### Pre-requisite files
The following files will be required during installation of Stellar Core:      
* Configuration file. Copy the example configuration file for a private network from here to your desktop: 
* Serivce file. Copy the stellar service file from here:      

### Stellar Core download and installation
* Download Stellar Core source code from GitHub: https://github.com/stellar/stellar-core/releases
* Follow the install instructions here: https://github.com/stellar/stellar-core/blob/master/INSTALL.md

### Files set-up
The following updates to the configuration file apply to all instances of the network.

* Update network pass-phrase
    * Change the value of `NETWORK_PASSPHRASE` to any phrase of your liking.

* Update database setting
    * Change the value of `DATABASE` to `sqlite3:///home/ubuntu/stellardb` If the database instance is a PostgreSQL instance (local or remote), then the value needs to be changed along the lines of database URI naming convention. For more information on database URI naming convention, refer (https://www.postgresql.org/docs/9.2/static/libpq-connect.html)

* Create directory for storing configuration file: `mkdir stellar-conf`
* Use SFTP client to put files:
    * Stellar configuration file to `/home/ubuntu/stellar-conf`
    * Stellar service file to `/home/ubuntu`
    * Copy stellar service file: `sudo cp /home/ubuntu/stellar.service /etc/systemd/system/stellar.service`

## Cloning VM
At this point, you have a Ubuntu-16.04 LTS VM with:
* Stellar Core installation 
* Local installation of `SQLite3` or `PostgreSQL`
* Stellar configuration file with common updates installed.

Stop this VM and create an image (Amazon Machine Instance - AMI) so that, it can be used to launch multiple instances.

# Set-up notes
With the installation in place, the configuration file needs to be updated for the private network.

## Configuration file updates     
To apply the following updates, it is important to run *all* the instances that is targeted for this network. To run instances, use the AMI that was created in an earlier step.

* Update list of peers in the network
    * Update the list of `KNOWN_PEERS` with the IP addresses of all the instances that _this instance is expected to connect to._ For the sake of simplicity, just add to this list the IP addresses of all instances in your network.

* Update peer connections information
    * Update `MAX_PEER_CONNECTIONS` to a number that represents all instances on your network. It is assumed that, the total number of instances in the network is not more than 10. Of course, some trial and error can be done to establish the optimal number.
    * Update `TARGET_PEER_CONNECTIONS` to around 80% of the value set for `MAX_PEER_CONNECTIONS`.
    * Update the list of `PREFERRED_PEERS` with the IP addresses of all instances that _this instance would prefer to connect to_. The number of such peers should be less than or equal to that set for `TARGET_PEER_CONNECTIONS`. While this article does not mandate a particular means to select such peers, in a production scenario some rules may apply for this selection.

* Update the list of `NODE_NAMES`
    * Use the public key (as generated in the previous step), and a easy to remember string to create a pair of values for the list of `NODE_NAMES`.
        
* Update validation settings
    * Ensure that `NODE_IS_VALIDATOR` is set to `true`.
    * Update the list of validators using the names that was set in `NODE_NAMES`. Note that, the number of validators is dependent on number of nodes that can fail. This article assumes a total of 10 instances in the network and therefore, the number of validators is set to 5.
* Update `NODE_SEED`
    * Generate node id by running this command: `stellar-core --genseed`
    * Save seed details - public and secret - use secret key for `NODE_SEED`

# Stellar Core start-up
To start-up Stellar first time ever, the database should be initialised. Then, Stellar can be started with the configuration file that was created earlier.

* Initialise database with this command: `stellar-core --conf /home/ubuntu/stellar-conf/private.cfg --newdb`
* Start Stellar with this command: `stellar-core --conf /home/ubuntu/stellar-conf/private.cfg`

To start Stellar on VM start-up use the service file that was copied earlier. To enable the service run the following commands:
```bash
sudo systemctl enable stellar
```
