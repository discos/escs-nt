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
Kernel    2.6.18-398.el5PAE
hostname  escs.noto.ira.inaf.it
eth1      192.167.187.17
========= =====

.. _escs_os_installation:

OS installation
~~~~~~~~~~~~~~~

Escs is installed on a 2TB hard disk so partitioned::

        [root@escs ~]# fdisk -l

        Disco /dev/sda: 2000.3 GB, 2000398934016 byte

        255 heads, 63 sectors/track, 243201 cylinders
        Unità = cilindri di 16065 * 512 = 8225280 byte

        Dispositivo Boot      Start         End      Blocks   Id  System
        /dev/sda1   *           1       63741   511999551   83  Linux
        /dev/sda2           63742       65781    16386300   82  Linux swap / Solaris
        /dev/sda3           65782      243201  1425126150   83  Linux

and::

        [root@escs ~]# mount
        /dev/sda1 on / type ext3 (rw)
        /dev/sda3 on /archive type ext3 (rw)

.. _escs_provisioning:

Provisioning
~~~~~~~~~~~~

Packages, users, groups and ACS are installed using `BASIE <http://github.com/discos/basie>`_ provisioning scripts.

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
connecting to the station server **192.167.187.78**::

    [root@escs /]# mkdir /var/log/ntpstats
    [root@escs /]# chown ntp:ntp /var/log/ntpstats
    [root@escs /]# service ntpd start
    [root@escs /]# chkconfig ntpd on

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
    

