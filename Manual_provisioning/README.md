# VagrantProject
Manual Provisioning

1. Bring up the VM - vagrant up

**Provisioning**
1. Nginx - web service
2. Tomcat - app service
3. RabbitMQ - Broker/Queuing Agent
4. Memcache - DB caching
5. ElasticSearch - Indexing/Search engine
6. MySQL - SQL database
   
**MySQL Setup:**
1. Login to the db vm - vagrant ssh db01
2. Verify Hosts entry - cat /etc/hosts
3. Update OS with latest patches:
- sudo mv /etc/yum.repos.d/fedora-updates.repo /tmp/
- sudo mv /etc/yum.repos.d/fedora-updates-modular.repo /tmp/
- sudo yum clean all
- sudo yum update -y
4. Install Maria DB package:
- sudo yum install git zip unzip -y
- sudo yum install mariadb-server -y
5. Start and enable mariadb-server
- systemctl start mariadb
- systemctl enable mariadb
6. mysql_secure_installation
7. Set DB name and users.
- mysql -u root -padmin123
- mysql> create database accounts;
- mysql> grant all privileges on accounts.* TO 'admin'@’%’ identified by 'admin123' ;
- mysql> FLUSH PRIVILEGES;
- mysql> exit;
8. Download Source code & Initialize Database
- git clone -b git@github.com:alishamaryangalan/VagrantProject.git
- cd VagrantProject
- mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
- mysql -u root -padmin123 accounts
- mysql> show tables;

9. Restart mariadb-server
- systemctl restart mariadb

**MEMCACHE SETUP**
1. Install, start & enable memcache on port 11211
- mv /etc/yum.repos.d/fedora-updates.repo /tmp/
- mv /etc/yum.repos.d/fedora-updates-modular.repo /tmp/
- yum clean all
- yum update -y
- sudo yum install memcached -y
- sudo systemctl start memcached
- sudo systemctl enable memcached
- sudo systemctl status memcached
2. Config change so memcache can listen remotely from any IP
- sed -i 's/OPTIONS="-l 127.0.0.1"/OPTIONS=""/' /etc/sysconfig/memcached
- sudo systemctl restart memcached
- sudo memcached -p 11211 -U 11111 -u memcached -d

**RABBITMQ SETUP**
1. Login to the RabbitMQ vm
- vagrant ssh rmq01
2. Verify Hosts entry, if entries missing update the it with IP and hostnames
- cat /etc/hosts
3. Update OS with latest patches
- sudo mv /etc/yum.repos.d/fedora-updates.repo /tmp/
- sudo mv /etc/yum.repos.d/fedora-updates-modular.repo /tmp/
- sudo yum clean all
- sudo yum update -y
4. Selinux settings
- sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
- setenforce 0
5. Set connection to the repositories
- curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
- sudo yum clean all
- sudo yum makecache
- sudo yum install erlang -y
- curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh |
- sudo bash
6. Install RabbitMQ Server Package
- sudo yum install rabbitmq-server -y
- rpm -qi rabbitmq-server
7. Start & Enable Service
- sudo systemctl start rabbitmq-server
- sudo systemctl enable rabbitmq-server
- sudo systemctl status rabbitmq-server
- sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
- sudo rabbitmqctl add_user test test
- sudo rabbitmqctl set_user_tags test administrator
- nohup sleep 30 && reboot & echo "going to reboot now"

**TOMCAT SETUP**
1. Login to the tomcat vm
- vagrant ssh app01
2. Verify Hosts entry, if entries missing update the it with IP and hostnames
- cat /etc/hosts
- sudo mv /etc/yum.repos.d/fedora-updates.repo /tmp/
- sudo mv /etc/yum.repos.d/fedora-updates-modular.repo /tmp/
- sudo yum clean all
3. Install Dependencies
- dnf -y install java-11-openjdk java-11-openjdk-devel
- dnf install git maven wget -y
4. Change dir to /tmp
- cd /tmp/
5. Download Tomcat Package
- wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
- tar xzvf apache-tomcat-9.0.75.tar.gz
6. Add tomcat user
- useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
7. Copy data to tomcat home dir
- cp -r /tmp/apache-tomcat-9.0.75/* /usr/local/tomcat/
8. Make tomcat user owner of tomcat home dir
- chown -R tomcat.tomcat /usr/local/tomcat
9. Setup systemctl command for tomcat service
10. Update file with following content.
- vi /etc/systemd/system/tomcat.service
  
[Unit]
Description=Tomcat
After=network.target
[Service]
User=tomcat
WorkingDirectory=/usr/local/tomcat
Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINE_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
SyslogIdentifier=tomcat-%i
[Install]
WantedBy=multi-user.target

- systemctl daemon-reload
- systemctl start tomcat
- systemctl enable tomcat

**CODE BUILD & DEPLOY (app01)**
1. Download Source code
- git clone -b main git@github.com:alishamaryangalan/VagrantProject.git
2. Update configuration
- cd VagrantProject
- vim src/main/resources/application.properties
3. Update file with backend server details
4. Build code
5. Run below command inside the repository (VagrantProject)
- mvn install
6. Deploy artifact
- systemctl stop tomcat
- rm -rf /usr/local/tomcat/webapps/ROOT*
- cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
- systemctl start tomcat
- chown tomcat.tomcat usr/local/tomcat/webapps -R
- systemctl restart tomcat

**NGINX SETUP**
1. Login to the Nginx vm
- vagrant ssh web01
2. Verify Hosts entry, if entries missing update the it with IP and hostnames
- cat /etc/hosts
3. Update OS with latest patches
- apt update
- apt upgrade
4. Install nginx
- apt install nginx -y
5. Create Nginx conf file with below content
- vi /etc/nginx/sites-available/vproapp

upstream vproapp {
server app01:8080;
}
server {
listen 80;
location / {
proxy_pass http://vproapp;
}
}

6. Remove default nginx conf
- rm -rf /etc/nginx/sites-enabled/default
7. Create link to activate website
- ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
8. Restart Nginx
- systemctl restart nginx