# VagrantProject

- Performed automated, repeatable, IAAC 

Tools required:
1. Hypervisor - Oracle VM VirtualBox
2. Automation - Vagrant
3. CLI - Git Bash
4. IDE - VisualStudio

Objective:
1. VM automation locally.

Architecture of project services: Nginx, Tomcat, RabbitMQ, Memcached, mysql
Architecture of automated setup: Vagrant, Virtualbox, GitBash

Stack: Collection of services working together to create an experience.

Nginx: Service used to create load balancing experience, configured to route the request to the Tomcat server.
Tomcat: Java web application service. NFS service for external storage. If there is a cluster of servers and you need a shared storage, we have NFS.

RabbitMQ: Its connected to Tomcat, its a message broker/queuing agent to connect 2 applications together.

User details are stored in mysql.

The request to access user details will first go to memcached service. It's a database caching, which is connected to MySQL server. When a request is first sent to db, db server sends it to Tomcat and then it will be cached in memcached server. 

Mysql: db server where all the data is stored
 

Automation:
Vagrant communicates with Oracle VM virtual box and creates VMs. Bash scripts, bash commands to set up services.

 