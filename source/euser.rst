.. _euser:

euser
-----

The euser machine is directly connected to a monitor in the control room and
is intended to be used by observers in order to run auxiliary observation tools
such as **quicklook** the **Imager** and **Basie**.

The machine main configuration parameters are the following: 

========= =====
PARAMETER VALUE
========= =====
hardware  Tower PC
CPU       Intel(R) Core(TM) i5-4690K CPU @ 3.50GHz
RAM       8GB
OS        CentOS Linux release 7.2.1511 (Core)
Kernel    3.10.0-327.36.3.el7.x86_64
hostname  euser.noto.ira.inaf.it
eth0      192.167.187.16
========= =====

USERS AND GROUPS
~~~~~~~~~~~~~~~~

Users and groups are defined so that data access is possible to the observers and according to 
the escs machine schema, this permits to mount NFS filesystems from escs machine without
issues. 

We have user **euser** belonging to group **acs**, and acs id is **335** 

NFS
~~~

The **/archive** filesystem from escs machine is mounted via NFS into this machine. 
The binding is realized via fstab:: 

        192.167.187.17:/exports/archive /archive        nfs     rw,soft,auto,intr,proto=tcp,port=2049,users,exec     0 0


IDL
~~~

This is the only machine running **IDL 8.3**  in this setup. IDL is installed via
original CD-ROM available at Medicina station and install location is the one
suggested by default::

    /usr/local/exelis

We then created a directory for IDL third party libraries in:: 

    /usr/local/idllib

and installed there **Coyote** and **astron** libraries. You can find original
versions of the libraries at the following links:

    * http://www.idlcoyote.com/programs/zip_files/coyoteprograms.zip
    * http://idlastro.gsfc.nasa.gov/ftp/astron.tar.gz

Remember to add th edirectory to the IDL path and to include all subdirectories
recursively, the easiest way to do that is via idlde->window->preferences.


SDI and Quicklook
~~~~~~~~~~~~~~~~~

The *Single dish imager* can be downloaded from the SRT scicom wiki page at
http://scicomsrt.pbworks.com/w/page/54294508/IMAGING%20ANALYSIS and has been
extracted to the same location in::

    /usr/local/idllib/SDI

SDI has **ds9** as a dependency, which can be download from
http://hea-www.harvard.edu/RD/ds9/site/Download.html or you can find in
/root/software/ directory. Extract ds9 and copy it in::

    /usr/local/bin

You can find the **fits_look.pro** idl procedure in /root/software directory. 
The procedure has been copied in */usr/local/idllib/* as the rest of custom idl
programs.

