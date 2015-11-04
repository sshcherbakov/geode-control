# geode-control
A set of bash scripts to simplify control of a Apache Geode cluster

## The Purpose

This set of scripts provides a consistent approach for starting Apache Geode cluster nodes.

With this set of scripts you can:

* Quickly start the cluster of Apache Geode nodes on a freshly provisioned machines
* Remotely control Geode cluster from a single node
	- start / stop cluster
	- centrally apply configuration changes to the cluster nodes using trivial template mechanism
	- gather logs and statistics files
	- clean up working directories
* Have coherent folder and configuration files layout
* Have startup script and configuration files templates


## Prerequisites

The bash scripts are primarily designed and tested on RHEL/CentOS 6.5.

The desired Java runtime must be pre-installed and available at the Bash prompt.

A copy of Apache Geode / Gemfire installation at arbitrary file system location with Geode / Gemfire `bin`
folder added to the `PATH` environment variable.

Password-less SSH authentication between the "master" and other cluster machines.

## Usage

Create a folder at the arbitrary file system location better ouside of any Geode/Gemfire installation.
It is convenient to have a stable symlink to the effective Geode/Gemfire installation folder, e.g.
````
ls -l /opt
lrwxrwxrwx.  1 root root   91 Oct 28 17:23 gemfire -> Pivotal_GemFire_8201_b17966_Linux/
drwxr-xr-x. 11 root root 4096 Oct 23 17:26 Pivotal_GemFire_8201_b17966_Linux
````

Copy or checkout the current repository to an arbitrary folder in your file system where the cluster instances 
will be based e.g.
````
cd ~
git clone https://github.com/sshcherbakov/geode-control.git
cd ~/geode-control
````

Create empty folder at the same location (`~/geode-control`) on all machines that will host the cluster.
The current user must have same read/write permissions to that folder on each of the machines.

Set the password-less authentication between the current machine (the master one) and other machines where you 
plan to run Geode / Gemfire nodes. [Here](http://hortonworks.com/kb/generating-ssh-keys-for-passwordless-login/) is an example how to do that.

Modify the `config/hostlist` describing the hosts where the cluster members are going to run. Each line of the file represents one Geode/Gemfire process instance and is specified in the following format:
````
<target_hostname> <locator|server> <instance_name> 
````

The `target_hostname` here is the name that will be used by the script to login via SSH remotely.
The `locator` or `server` parameter specifies which kind of Geode/Gemfire node to start respectively.
The `instance_name` is the name of the Geode/Gemfire process instance that will also be used as a folder name for the
instance related files.


Place custom Jar files that need to be available at the Geode/Gemfire classpath at runtime to the optional `lib/` subfolder.

We are basically set now. The following commands will bring the Geode cluster up and running.

### Configuration deployment

````
bin/cluster deploy
````
copies the contents of `bin/`, `config/` and `lib/` subfolders to the cluster member machines as specified in the `config/hostlist` file.
The placeholders in the property files get substituted with the local environment variables on each host accordingly.

### Cluster start

````
bin/cluster start
````

Starts all cluster nodes


### Cluster stop

````
bin/cluster stop
````

Starts all running cluster nodes

### Clean up log and work folder

````
bin/cluster clean
````

Removes all log and Geode/Gemfire work files located in the `var/` subfolder on each machine.


### Gather logs and stats files

````
bin/cluster logs
````

Gathers all Geode/Gemfire logs and statistics files from all cluster machines into a singe compressed time stamped tar ball file
placed in the current directory. The file contents is flattened with the log and stats file names indicating the originating Geode/Gemifre instance.



## Folder structure
````
root@vf0 geode-control]# tree
.
├── bin 						<-- geode-control scripts
│   ├── cluster 				<-- main script for the cluster control
│   ├── start-locator 			<-- includes process environment variables
│   ├── start-server 			<-- includes process environment variables
│   ├── stop-locator
│   └── stop-server
├── config 						<-- configuration file templates for all cluster participants
│   ├── cache.xml
│   ├── gemfire-locator.properties
│   ├── gemfire-server.properties
│   └── hostlist
├── etc 						<-- runtime configuration
│   ├── locator1 				<-- runtime Gode/Gemfire instance configuration with subsituted placeholders
│   │   └── gemfire.properties
│   └── server1 				<-- runtime Gode/Gemfire instance configuration with subsituted placeholders
│       ├── cache.xml
│       └── gemfire.properties
├── lib 						<-- custom Jars to add to each cluster instance classpath at runtime
│   ├── my.jar
└── var
    ├── log 					<-- Geode/Gemfire instance logs
    │   ├── locator1
    │   │   ├── locator1-vf0.log
    │   │   └── meta-locator1-vf0-01.log
    │   └── server1
    │       ├── meta-server1-vf0-01.log
    │       ├── server1-vf0-01-00.marker
    │       ├── server1-vf0.gfs
    │       └── server1-vf0.log
    └── work 					<-- Geode/Gemfire work folders
        ├── locator1
        │   ├── ConfigDiskDir_locator1
        │   │   ├── BACKUPcluster_config_1.crf
        │   │   ├── BACKUPcluster_config_1.drf
        │   │   ├── BACKUPcluster_config.if
        │   │   └── DRLK_IFcluster_config.lk
        │   ├── GemFire_root
...
        │   ├── locator10334state.dat
        │   ├── locator10334views.log
        │   └── vf.gf.locator.pid
        └── server1
            └── vf.gf.server.pid
````

After deployment and startup step the file structure is identical on all cluster machines.

