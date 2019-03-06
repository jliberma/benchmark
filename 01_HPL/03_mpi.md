Install and Test MPI
===================

An MPI is required for distributed memory tests. These are HPL runs that span more than one physical server connected via a network.

There are many commercial and opens ource MPI implementations available.

In this example we use [MPICH 1.3][1], which conforms to the MPI-3 standard.

Install MPICH
============

This process must be repeated on all nodes.

~~~
$ cd $HOME
$ wget http://www.mpich.org/static/downloads/3.3/mpich-3.3.tar.gz
$ tar zxvf mpich-3.3.tar.gz 
$ mkdir mpich3-build
$ mkdir mpich3-install
$ cd mpich3-build/
$ ../mpich-3.3/configure --prefix=$HOME/mpich3-install/
$ make -j 8
$ make -j 8 install
$ cd ../mpich3-install/
$ export PATH=$PATH:$HOME/mpich3-install/bin
$ echo $PATH
$ which mpicc
/root/mpich3-install/bin/mpicc
$ mpicc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-36)
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
~~~

Configure Password-less SSH (or RSH)
===================================

Make sure you can access all systems remotely.

~~~
$ ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.

$ ls -al ~/.ssh/id_rsa*
-rw-------. 1 root root 1679 Mar  5 11:38 /root/.ssh/id_rsa
-rw-r--r--. 1 root root  422 Mar  5 11:38 /root/.ssh/id_rsa.pub

$ ssh-copy-id 192.168.122.33
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
root@192.168.122.33's password: 

Number of key(s) added: 1

$ ssh 192.168.122.33 uptime
 11:39:34 up 4 days, 19:43,  0 users,  load average: 0.00, 0.01, 0.05
~~~

Prepare the Environment
===============

Add the path to the MPI executables to the environment.

~~~~~
$ grep PATH ~/.bash_profile
PATH=$PATH:$HOME/bin:$HOME/mpich3-install/bin
export PATH

$ source ~/.bash_profile

$ which mpiexec
/root/mpich3-install/bin/mpiexec
~~~~~~


Test MPICH
==========

~~~
$ cat << EOF > ~/machinefile
192.168.122.33
192.168.122.34
EOF

$ mpiexec -f ~/machinefile -np 16 hostname
node13
node13
node13
node13
node13
node13
node13
node13
node14
node14
node14
node14
node14
node14
node14
node14

$ mpiexec -f ~/machinefile -np 16  ~/mpich3-build/examples/cpi
Process 5 of 16 is on node14
Process 13 of 16 is on node14
Process 15 of 16 is on node14
Process 4 of 16 is on node13
Process 0 of 16 is on node13
Process 7 of 16 is on node14
Process 2 of 16 is on node13
Process 1 of 16 is on node14
Process 6 of 16 is on node13
Process 3 of 16 is on node14
Process 10 of 16 is on node13
Process 9 of 16 is on node14
Process 12 of 16 is on node13
Process 11 of 16 is on node14
Process 14 of 16 is on node13
Process 8 of 16 is on node13
pi is approximately 3.1415926544231274, Error is 0.0000000008333343
wall clock time = 0.000711
~~~

NOTE: You may have to change firewalld or iptables rules to allow RELATED,ESTABLISHED connections.

[1][https://www.mpich.org/]
