.. _escs:

escs
----

This machine is the main responsible of the running system and the most critical possible point of failure.
It is its responsibility to:

* run the ACS framework and the control system
* export data via NFS


========= =====
PARAMETER VALUE
========= =====
hardware  PC Tower 
CPU       Intel(R) Quad Core(TM) i3 CPU 540 @ 3.07GHz
RAM       8GB
OS        Scientific Linux 5.3 i386 (32 bit)
Kernel    2.6.18-128.1.1.el5PAE
hostname  escs.noto.ira.inaf.it
eth0      192.167.187.17
========= =====

.. _escs_os_installation:

OS installation
~~~~~~~~~~~~~~~

Escs is installed on a 2TB hard disk so partitioned::

   [root@escs /]# fdisk -l

   Disk /dev/sda: 500.1 GB, 500107862016 bytes
   255 heads, 63 sectors/track, 60801 cylinders
   Units = cylinders of 16065 * 512 = 8225280 bytes

   Device Boot      Start         End      Blocks   Id  System
   /dev/sda1   *           1          13      104391   83  Linux
   /dev/sda2              14        2624    20972857+  83  Linux
   /dev/sda3            2625        3668     8385930   82  Linux swap / Solaris

and::

   [root@escs /]# mount

   /dev/sda6 on /system type ext3 (rw)

.. _escs_users:

Users and Groups
~~~~~~~~~~~~~~~~

Defined groups are:

========== === ==========
group name gid group role
========== === ==========
escs       335 owns ACS system files and processes
observers  336 final user accounts which run observation tools
========== === ==========

While necessary users are:

========= ====  ============== =========
user name uid   groups         user role
========= ====  ============== =========
manager   3060  observers,escs Run the ACS system
observer  3061  escs           Executes the observations
========= ====  ============== =========

Create those with::

    [root@escs /]# groupadd -g 335 escs
    [root@escs /]# groupadd -g 336 observers
    [root@escs /]# useradd -g observers -G escs -n -u 3060 manager
    [root@escs /]# useradd -g escs -n -u 3061 escs

    /etc/sudoers /etc/shutodwn.allow /etc/inittab /etc/pam.d/login
    /etc/pam.d/sshd /etc/security/access.conf /etc/skel/.bashrc
    /etc/skel/.bash_profile /etc/skel/.idl

With this files, users are prohibited from shutting down the machine or putting it
offline and reboot. SSH login is permitted only to root and observers group in
order to run observations.

Pam files are removed for preventing accidental shutdowns::

    [root@escs /]# rm /etc/security/console.apps/poweroff
    [root@escs /]# rm /etc/security/console.apps/halt
    [root@escs /]# rm /etc/security/console.apps/reboot

Then we execute *gdmsetup* in order to disable login window actions and
configure the welcome message::

    [root@escs /]# gdmsetup

.. _escs_provisioning:

Provisioning
~~~~~~~~~~~~

Packages are installed using `BASIE <http://github.com/discos/basie>`_ provisioning scripts.

ACS software package must be configured to run on the escs node. We first
extract necessary files and then configure users to load the correct environment
variables::

    [root@escs /]# mkdir alma
    [root@escs /]# chown manager:escs /alma
    [root@escs /]# su - manager
    escs manager:~ > cd /
    escs manager:/ > tar xzpvf /home/manager/ACS.tar.gz
    escs manager:/ > cd alma; chown manager:escs ACS-8.2/
    escs manager:/alma > cp -r /alma/ACS-8.2/ACSSW/config/.acs $HOME
    escs manager:/ > vi ~/.bashrc
    escs manager:/ > vi ~/.bash_profile
    escs manager:/ > su - observer
    escs observer:/ > vi ~/.bashrc
    escs observer:/ > vi ~/.bash_profile
    
We create the necessary directories and set the right permissions::

    [root@escs /]# chown manager:escs system
    [root@escs /]# chown manager:observers archive
    [root@escs /]# mkdir /system/configuration
    [root@escs /]# chown manager:escs /system/configuration
    [root@escs /]# mkdir /system/docroot
    [root@escs /]# chown manager:escs /system/docroot
    [root@escs /]# mkdir /system/introot
    [root@escs /]# chown manager:escs /system/introot
    [root@escs /]# mkdir /system/sources
    [root@escs /]# chown manager:escs /system/sources
    [root@escs /]# mkdir /system/userbin
    [root@escs /]# chown manager:observers /system/userbin
    [root@escs /]# su - manager
    escs manager:~ > cd /archive
    escs manager:/archive > mkdir /archive/data
    escs manager:/archive > mkdir /archive/schedules
    escs manager:/archive > mkdir /archive/logs
    escs manager:/archive > mkdir /archive/events
    escs manager:/archive > mkdir /archive/extraData
    escs manager:/archive > chmod 710 /archive/*

Then we can checkout and install the escs system::

   [root@escs /]# chmod a+rw /data
   [root@escs /]# cd data
   [root@escs /data]# mkdir ACS
   [root@escs /data]# chown manager:escs ACS
   [root@escs /data]# su - manager
   escs manager:/ > cd /data/ACS
   escs manager:/data/ACS> svn co http://belzebu.oa-cagliari.inaf.it/repos/ACS .
   escs manager:/data/ACS > cd /data/ACS/trunk/SystemMake #this will change to  ACS/tags/escs-0.3
   escs manager:/data/ACS/trunk/SystemMake > make all
   escs manager:/data/ACS/trunk/SystemMake > make cdb
   escs manager:/data/ACS/trunk/SystemMake > escsInstall

And we can set acs to start at boot time::

   [root@escs /]# vim /etc/rc.local
   su -l manager -c acsservicesdaemon &
   su -l manager -c acscontainerdaemon &

.. _escs_temporary_data:

ACS Temporary Data
~~~~~~~~~~~~~~~~~~

ACS needs to store log informations for each process running inside the system.
this is true for every container, daemon, manager ecc... 
This files can be very large and sometimes they can flood the disk space
resulting in wrong ACS behaviors, so we decided to store these files into a
local directory on each machine::

    [manager@escs ~] vim ~/.bashrc
    ...
    export ACS_TMP=/data/ACSTMP

And we create the necessary directory setting owner and group to the ones used
by ACS processes::

    [manager@escs ~] cd /data
    [manager@escs data/] mkdir ACSTMP
    [manager@escs data/] chown manager:escs ACSTMP


.. _escs_ntp:

NTP
~~~

Ntp service for system clock synchronization is configured via */etc/ntp.conf*
and */etc/ntp/ntpservers*
connecting to the station server **192.167.187.78**.
We also define a custom */root/bin/plot_loopstats* command::

    [root@escs /]# mkdir /var/log/ntpstats
    [root@escs /]# chown ntp:ntp /var/log/ntpstats
    [root@escs /]# service ntpd start
    [root@escs /]# chkconfig ntpd on
    [root@escs /]# /root/bin/plot_loopstats

NFS
~~~

NFS is used in order to export archived data.

First, we create the exported filesystem directories::

    [root@escs /]# mkdir /exports
    [root@escs /]# mkdir /exports/archive

Then we bind the filesystem to the exported directories modifying the
*/etc/fstab* file adding the following lines::

   /archive                  /exports/archive           none    bind            0 0

Now the OS must be instructed to export the bound filesystems::

    [root@escs /]# cat /etc/exports
    /exports         192.167.187.0/24(rw,fsid=0,insecure,no_subtree_check,sync,no_root_squash)
    /exports/archive    192.167.187.0/24(rw,nohide,insecure,no_subtree_check,sync,no_root_squash)
    [root@escs /]# exportfs -rv

And we start the nfs server::
 
    [root@escs /]# service nfs start
    [root@escs /]# chkconf nfs on

DATA BACKUP
~~~~~~~~~~~

Data backup is realized on IRA-Bologna servers via rsync. we thus must authorize
IRA server to use rsync service on escs which is the public access point
of the control system and enable rsync service on the machine itself::

    [root@escs /]# vim /etc/rsyncd.conf
    [Area-Med-Arc]
        comment=archivio osservazioni single dish di Noto
        path=/archive/data
        read only = yes
        list = yes
        host allow =
        192.167.165.0/255.255.255.0
        uid = 3060
        gid = 335

    [root@escs /]# vim /etc/xinetd.d/rsync
    # default: off
    # description: The rsync server is a good addition to an ftp server, as it \
    #       allows crc checksumming etc.
    service rsync
    {
        disable = no
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/bin/rsync
        server_args     = --daemon
        log_on_failure  += USERID
    }

Service can be started and monitored using::

    [root@escs /]# service xinted start|stop|status|restart
    

