# vagrant-hadoop-cluster - extended

Deploying [apache-hadoop](http://hadoop.apache.org) in a virtualized cluster as
easy as 1-2-3. Have your own cluster for local testing or just counting words.

This work is based on:
http://cscarioni.blogspot.co.uk/2012/09/setting-up-hadoop-virtual-cluster-with.html

## Deploying the cluster

Simply clone the repository and install
[vagrant](http://downloads.vagrantup.com/) for your platform. Change into the
directory and run the following:

    $ vagrant box add base-hadoop http://files.vagrantup.com/lucid64.box
    $ vagrant up

This will set up 4 machines - `master`, `hadoop1`, `hadoop2` and `hadoop3`. Each 
of them will have two CPUs and 1GB of RAM. If this is too much for your machine, 
adjust the `Vagrantfile`.

The machines will be deployed using [pupptet](http://puppetlabs.com/). All of them
will have hadoop (apache-hadoop-1.1.2) installed, ssh will be configured and
local name resolution also works. 

Hadoop is installed in `/opt/hadoop-1.1.2` and all tools are in the `PATH`.

The `master` machine acts as the namenode and jobtracker, the 3 others are data
nodes and task trackers.

### Starting the cluster

This cluster uses the `ssh-into-all-the-boxes-and-start-things-up`-approach,
which is fine for testing. Also for simplicity, everything is running as `root`
(patches welcome).

Once all machines are up and provisioned, the cluster can be started. Log into
the master, format hdfs and start the cluster.

     $ vagrant ssh master
     $ (master) sudo hadoop namenode -format
     $ (master) sudo start-all.sh

After a little while, all daemons will be running and you have a fully working
hadoop cluster.

### Stopping the cluster

If you want to shut down your cluster, but want to keep it around for later
use, shut down all the services and tell vagrant to stop the machines like this:

     $ vagrant ssh master
     $ (master) sudo stop-all.sh
     $ exit or Ctrl-D
     $ vagrant halt

When you want to use your cluster again, simply do this:

     $ vagrant up
     $ vagrant ssh master
     $ (master) sudo start-all.sh


### Getting rid of the cluster

If you don't need the cluster anymore and want to get your disk-space back do
this:

     $ vagrant destroy

This will only delete the VMs all local files in the directory stay untouched
and can be used again, if you decide to start up a new cluster.
     

## Interacting with the cluster

### Webinterface

The namenode webinterface is available under http://master.local:50070/dfshealth.jsp and the
jobtracker is available under http://master.local:50030/jobtracker.jsp

The cluster uses [zeroconf](http://en.wikipedia.org/wiki/Zero-configuration_networking) 
(a.k.a. bonjour) for name resolution. This means, that
you never have to remember any IP nor will you have to fiddle with your
`/etc/hosts` file.

Name resolution works from the host to all VMs and between all VMs as well.  If
you are using linux, make sure you have `avahi-daemon` installed and it is
running. On a Mac everything should just work (TM) witouth doing anything. 
(Windows testers and patches welcome).

The network used is `192.168.7.0/24`. If that causes any problems, change the
`Vagrantfile` and `modules/avahi/file/hosts` files to something that works for
you. Since everything else is name based, no other change is required.

### Command line

To interact with the cluster on the command line, log into the master and
use the hadoop command as `root`(again, patches welcome).

    $ vagrant ssh master
    $ (master) sudo -i
    $ (master) hadoop fs -ls /
    $ ...

You can access the host file system from `/vagrant`, which means you can drop
your hadoop job in there and run it on your own fully distributed hadoop
cluster.

## Performance

Since this is a fully virtualized environment running on your computer, it will
not be super-fast. This is not the goal of this setup. The goal is to have a fully
distributed cluster for testing and troubleshooting. 

To not overload the host machine, has each tasktracker a hard limit of 1 map task
and 1 reduce task at a time. 


## Cascading SDK

The project supports deploying the [Cascading SDK](http://cascading.org/sdk) on
the cluster as well. If you want that feature, remove the comment in
`manifests/base-hadpp-pp` line 3. This will download the SDK and put all SDK
tools into the `PATH`. The SDK itself can be found in `/opt/CascadingSDK`.

## Hacking & Troubleshooting

### Slow download

The puppet module is fetching the hadoop distribution from a hard-coded mirror.
This is due to technical reasons with the mirror system of apache and out of my
control.  If the mirror used is too slow for you, just point the URL in
`/modules/hadoop/manifests/init.pp` line 5 to an
[apache-mirror](http://www.apache.org/dyn/closer.cgi) closer to you.

### Puppet

If you change any of the puppet modules, you can simply apply the changes with
vagrants built-in provisioner.

    $ vagrant provision

## Wishlist

- have it working on windows
- run as other user than root
- have a way to configure the names/ips in only one file
- have a way to define the hadoop version globally
- use dynamic apache mirror for downloading
