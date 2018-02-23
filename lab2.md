## Overview

This lab session is a low-level, hands-on introduction to container security using Red Hat Enterprise Linux 7. It can be delivered by an instructor or consumed as a series of self paced exercises.

### Prerequisites

* Fundamental Red Hat Enterprise Linux user and administrator concepts. 
* Basic text editing skills using vim or nano.
* An introductory knowledge of Docker is helpful.

#### Lab Enviroment

You will be working with the following RHEL7.4 Server systems. 

* {{SERVER_0}} (Container Host)
* {{SERVER_1}}  (Container Registry)
* {{SERVER_2}} (Container Registry)
* {{SERVER_DIST}} (Content Server)

Login for all servers: User: {{ROOT_USERNAME}} Password: {{ROOT_PASSWORD}}

To prepare for the lab, use an ssh client and login to {{SERVER_0}}, {{SERVER_1}} and {{SERVER_2}}.

Example

~~~shell
$ ssh {{ROOT_USERNAME}}@{{SERVER_0}}
~~~

