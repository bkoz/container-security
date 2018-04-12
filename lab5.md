## Isolation

### Overview
Containers provide a certain degree of process isolation via kernel namespaces. In this lab, we’ll examine the capabilities of a process running in a containerized namespace. We’ll begin by running a container and looking at it’s capabilities.

### Capabilities

We’ll begin with looking at Linux capabilities as it relates to containers. Capabilities are distinct units of privilege that can be independently enabled or disabled. Start by examining the kernel header file and the effective capabilities of a root process on {{SERVER_0}} by looking its status. Notice that all 37 capability bits are set indicating this process has a full set of capabilities. For more info, read [Dan’s blog post](http://rhelblog.redhat.com/2016/10/17/secure-your-containers-with-this-one-weird-trick/). 

~~~shell
# yum -y install kernel-headers
# less /usr/include/linux/capability.h
~~~

Examine the capabilities of a process running as root on the host.

~~~shell
# grep CapEff /proc/self/status

CapEff:	0000001fffffffff
~~~

The ```capsh``` and ```pscap``` commands provide a human readable output of the capabilities bitmask. Try it out!

~~~shell
# yum -y install libcap-ng
# capsh --decode=01fffffffff

0x0000001fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,35,36
~~~

Exploring the capabilities of containers.

Start by loading the rhel7 image.

~~~shell
wget -O - http://{{SERVER_DIST}}/content/images/rhel7.tar | docker load
~~~

Run a rhel7 container as user 0 and look at it’s capabilities. A non-null CapEff value indicates the process has capabilities. Take note that the capabilities are less than what a root process has running on the host.

~~~shell
# docker run --rm -it rhel7 grep CapEff /proc/self/status

CapEff:	00000000a80425fb
~~~

Now run the same container as a non-root user and compare the results to the previous exercises.

~~~shell
# docker run --rm -it --user 32767 rhel7 grep CapEff /proc/self/status

CapEff:	0000000000000000
~~~

Next, run the same container as privileged and compare the results to the previous exercises. What conclusions can you draw?

~~~shell
# docker run --rm -it --privileged rhel7 grep CapEff /proc/self/status

CapEff: 0000001fffffffff
~~~

Next, run the container as root but drop all capabilities.

~~~shell
# docker run --rm -ti --user 0 --name temp --cap-drop=all rhel7 grep CapEff /proc/self/status

CapEff:	0000000000000000
~~~

Now, run the container as root but add all capabilities.

~~~shell
# docker run --rm -ti --user 0 --name temp --cap-add=all rhel7 grep CapEff /proc/self/status

CapEff: 0000001fffffffff
~~~

#### Capabilities Challenge #1

How could you determine which capabilities docker drops from a process running in a container? One solution is presented below.

One approach would be to use your favorite binary calculator and find the CapEff difference between a host process (0x1fffffffff) and a containerized process (0xa80425fb) then use capsh to decode it.

~~~shell
# yum -y install bc
# echo 'obase=16;ibase=16;1FFFFFFFFF-A80425FB' | bc
1F57FBDA04

# capsh --decode=1F57FBDA04 

0x0000001f57fbda04=cap_dac_read_search,cap_linux_immutable,cap_net_broadcast,cap_net_admin,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_lease,cap_audit_control,cap_mac_override,cap_mac_admin,cap_syslog,35,36
~~~

#### Capabilities Challenge #2

Let’s say you're working with a time/date sensitive application that gathers, logs and locks data. The application provider tells you this container requires full privileges because it needs to set a file as immutable (via the chattr command). You remember that in compliance with your company’s security policy, this container should not be able to ping any host. Your challenge is to run the application safely yet produce the (3) GOOD test results as shown below.

~~~shell
# Installing Application...
Fri Apr  7 21:41:49 UTC 2017
ping test fails: GOOD
chattr test: GOOD
----i--------e-- /var/tmp/timestamp
file immutable test: GOOD
~~~

Make sure the mystery container is loaded.

~~~shell
# wget -O - http://{{SERVER_DIST}}/content/images/mystery.tar | docker load
~~~

To get started, run the container and observe it produces several NOT GOOD messages. Use what you have learned so far about capabilities 
to determine the proper arguments to pass to ```docker run``` and solve the challenge.

~~~shell
# docker run --rm mystery
Installing Application...
Fri Apr  7 21:22:47 UTC 2017
ping works: NOT GOOD
chattr: Operation not permitted while setting flags on /var/tmp/timestamp
chattr failed: NOT GOOD
-------------e-- /var/tmp/timestamp
file is not immutable: NOT GOOD
~~~

WARNING: Recall the risks of running a privileged container? In order to complete your investigation of this container, you may 
consider running the mystery container as privileged. If you do so, observe the output carefully and run tail -f /var/log/messages 
to look for clues of an exploitation. You’ll need to perform a minor repair to your container host {{SERVER_0}} if you run
the mystery container with ```--privileged```.

Proceed with caution.

~~~shell
# docker run --rm --privileged mystery 
~~~
 
One solution is to add the linux_immutable and drop the net_raw capabilities.
 
~~~shell
# docker run --rm --cap-add=linux_immutable --cap-drop=net_raw mystery
~~~

An even better approach is to drop all capabilities and add only what is required.

~~~shell
# docker run --rm --cap-drop=all --cap-add=linux_immutable mystery
~~~

#### Capabilities Challenge #3

Suppose a container had a legitimate reason to change the date (ntpd, license testing, etc) How would you allow a container to change the date on the host? What capabilities are needed to allow this? One solution is below.

To allow a container to set the system clock, the ```sys_time capability``` must be added. Also, at the time of this writing, the seccomp security option must be set to unconfined. This is a known issue. Refer to the [RHEL7 release notes](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/release_notes/known_issues) for details.

~~~
# docker run --rm --cap-drop=all --cap-add=sys_time --security-opt=seccomp=unconfined mystery
~~~

Take note of the output and check the date on {{SERVER_0}}.

~~~
# date

Wed Dec 31 19:00:09 EST 1969
~~~

**IMPORTANT** => Make sure to reset the date correctly to prevent issues with future labs.

~~~shell
# date MMDDhhmmYYYY
~~~
