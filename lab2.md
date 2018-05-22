## Introduction

This lab session is a low-level, hands-on introduction to container security using Red Hat Enterprise Linux 7. It can be delivered by an instructor or consumed as a series of self paced exercises.

### Prerequisites

* Fundamental Red Hat Enterprise Linux user and administrator concepts. 
* Basic text editing skills using vim or nano.
* An introductory knowledge of Docker is helpful.

#### Lab Environment

![Lab Diagram]({% image_path con-sec-lab.png %})

You will be working with the following systems running Red Hat Enterprise Linux (RHEL) version 7.5. 

* {{SERVER_0}} (Container Host)
* {{SERVER_1}} (Container Registry)
* {{SERVER_2}} (Container Registry)
* {{SERVER_DIST}} (Bastion host and content server)
  * Repos
    * rhel-7-server-rpms 
    * rhel-7-server-optional-rpms 
    * rhel-7-server-extras-rpms 
    * rhel-7-server-supplementary-rpms
  * Content (container images, scanner example code) 

{{SERVER_0}}, {{SERVER_1}} and {{SERVER_2}} can only be accessed from a bastion host which is publicly accessible via ssh.

##### Lab Environment

##### DIY
You can easily build out this lab yourself. Just get a hold of (3) RHEL7.5 servers that are subscribed to the repos listed above. Contact bkozdemb@redhat.com for the image content (I can't host it to the general public ). The bastion host is optional.

##### Red Hat Internal
If you are internal to Red Hat, this lab and the content is available via the [RHPDS](https://rhpds.redhat.com/service/explorer). Make sure to upload your ssh key to RHPDS so you can use ```ssh``` to login the bastion host. Once your bastion host has been provisioned, you should receive an email from OPENTLC similar to the following.

~~~shell
Your Red Hat Product Demo System application RHPDS-RH-user-redhat.com-SUMMIT-L1007-GUID host {{BASTION}} can now authenticate. Take note that your lab environment may still have other hosts building, so please make sure they are all running before starting your demo/lab.
~~~

#### Exercise

Access the {{SERVER_0}}, {{SERVER_1}} and {{SERVER_2}} systems as follows.

1) Using an ssh client, substitute the global user id (**GUID**) and your RHPDS user-name to login to the bastion host.

~~~shell
$ ssh user-redhat.com@{{BASTION}}
~~~

2) ```sudo``` to the **cloud-user**

~~~shell
[user-redhat.com@bastion-GUID ~]$ sudo -iu cloud-user
~~~

3) Now, as **cloud-user**, login to the servers as {{ROOT_USERNAME}}.

~~~shell
[cloud-user@bastion-GUID ~]$ ssh {{ROOT_USERNAME}}@{{SERVER_0}}
~~~

