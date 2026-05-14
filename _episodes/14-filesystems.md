---
title: "Filesystems and Storage"
author: "Colin Sauze, Ed Bennett, Jarno Rantaharju, Thomas Green"
teaching: 15
exercises: 10
questions:
 - "Where can I store my data?"
 - "What is the difference between scratch and home filestore?"
 - "How to use filesystems in job scripts."
objectives:
 - "Understand the difference between home and scratch directories"
keypoints:
 - "The home directory is the default place to store data."
 - "The scratch directory is a larger space for temporary files."
 - "On Hawk in Cardiff home is backed up but is also a slower disk."
 - "Quotas on home are much smaller than scratch."
---


# Filesystems and Storage

## What is a filesystem?
Storage on most compute systems is not what and where you think they are! Physical disks are bundled together into a virtual volume; this virtual volume may represent one filesystem, or may be divided up, or partitioned, into multiple filesystems. And your directories then reside within one of these fileystems. Filesystems are accessed over the network through mount points.

There are multiple storage/filesystems options available for you to do your work. The most common are:

* home: Where you land when you first login. 50 GB per user. Slower access, backed up. Used to store your work long term. 
* scratch: Shared between all users of a project. Temporary working space. Lustre filesystem with usually faster access, not backed up. Larger quota, but old files might get deleted. DON'T STORE RESULTS HERE!


Here's a synopsis of filesystems on Falcon in Cardiff:

|Name|Path|Default Quota|Disk Size|Backed Up|Filesystem
|-----------------|---|----|-----|---|-----|------|
|Home|/shared/home1/user.name|50GB|98TB|No|NFS|
|Scratch|/scratch/SCWF00XXXX|3TB + 3million files|1300TB|No|Lustre|

**Important!! Ensure that you don't store anything longer than necessary on scratch, this can negatively affect other people’s jobs on the system.**


# Accessing your filestore

## How much quota do I have left on my home directory?

Login to a login node (e.g. `falconlogin.cf.ac.uk`) and run the ```myquota``` command. This will tell you how much space is left in your home directory.

~~~
$ myquota
~~~
{: .bash}

~~~

$ myquota

HOME quotas are set at 50 GB by default. You can find more information about
your current usage with the command `du --human-readable  --max-depth=1 $HOME`
------------------------------

SCRATCH PROJECT DIRECTORIES RELATING TO user PROJECT GROUPS
------------------------------
PROJECT SCRATCH DIR SCWF00003
      Filesystem    used   bquota  blimit  bgrace   files   iquota  ilimit  igrace
/scratch/SCWF00003  707.7G       0k      3T       -  196129        0 3000000       -

PROJECT SCRATCH DIR SCWF00004
      Filesystem    used   bquota  blimit  bgrace   files   iquota  ilimit  igrace
/scratch/SCWF00004  146.3M       0k      3T       -    4757        0 3000000       -

~~~
{: .output}


## How much scratch is available in the system?

The ```df``` command tells you how much disk space is left in a filesystem. The ```-h``` argument makes the output easier to read, it gives human readable units like M, G and T for Megabyte, Gigabyte and Terrabyte instead of just giving output in bytes. By default df will give us the free space on all the drives on a system, but we can just ask for the scratch drive by adding ```/scratch``` as an argument after the ```-h```.

~~~
$ df -h /scratch
~~~
{: .bash}

~~~
Filesystem                                                                                                                                                      Size  Used Avail Use% Mounted on
172.23.10.224@o2ib,172.23.10.228@o2ib:172.23.10.225@o2ib,172.23.10.229@o2ib:172.23.10.226@o2ib,172.23.10.230@o2ib:172.23.10.227@o2ib,172.23.10.231@o2ib:/exafs  1.3P  800T  431T  66% /shared/scratch
{: .output}

There is also a limit how many files can be stored on a filesystem.  These are called inodes and can be found with the ```-i``` argument to ```df```.  Usually not a problem but be careful if creating many files and talk to your local support or RSE if your code creates many files (in the 1000s).

More detailed information of Lustre usage can be found with the ```lfs``` command.  This shows that ```/scratch``` is a
parallel fileystem with many storage areas.  Usually a file is *striped* (divided) across these storage areas and provide faster
access especially with larger files.  This can be configured by the user if required.

~~~
$ lfs df -h /scratch
~~~
{: .bash}

~~~
UUID                       bytes        Used   Available Use% Mounted on
exafs-MDT0000_UUID        958.8G       20.2G      922.2G   3% /shared/scratch[MDT:0]
exafs-MDT0001_UUID        958.8G       15.6G      926.7G   2% /shared/scratch[MDT:1]
.... many lines omitted ....
exafs-OST0008_UUID        514.9T      390.3T       98.6T  80% /shared/scratch[OST:8]
exafs-OST0009_UUID        514.9T      390.3T       98.6T  80% /shared/scratch[OST:9]

filesystem_summary:         1.3P      799.6T      430.4T  66% /shared/scratch
~~~
{: .output}

# Filesystems in job scripts.

With home and scratch filesystems (along with standard local ```/tmp``` filesystem)
the most appropriate filesystem should be used in the job script.

When running jobs all files should normally be copied to ```/scratch``` before running the application.  This makes sure all
files are read on the high performance Lustre filesystem that ```/scratch``` uses.  For example, if ```my_job``` directory in
```$HOME``` contains all files for the job, copy to ```/scratch``` and then run application.
~~~
WDPATH=/scratch/SCWFXXXXX/$USER/$SLURM_JOB_ID
rm -fr $WDPATH
mkdir -p $WDPATH
cp -r $HOME/my_job $WDPATH

cd $WDPATH
my_application.exe
echo "INFO: Finished job in $WDPATH"
~~~
{: .bash}

# Exercises

> ## Using the `df` command. 
> 1. Login to a login node
> 2. Run the command `df -h`.
> 3. How much space does /scratch have left?
> 4. If you had to run a large job requiring 10TB of scratch space, would there be enough space for it?
{: .challenge}

> ## Using the `myquota` command.
> 1. Login to a login node.
> 2. Run the `myquota` command. 
> 3. How much space have you used and how much do you have left? 
> 4. If you had a job that resulted in 60GB of files would you have enough space to store them?
{: .challenge}


