# 1. Wildfly설치 - CentOS

## 1) Create a user and group for WildFly

    $ groupadd -r wildfly
    $ useradd -r -g wildfly -d /opt/wildfly -s /sbin/nologin wildfly

## 2) Download the Wildfly Installation file

    $ Version_Number=16.0.0.Final
    $ wget https://download.jboss.org/wildfly/$Version_Number/wildfly-$Version_Number.tar.gz -P /tmp
    $ tar xf /tmp/wildfly-$Version_Number.tar.gz -C /opt/

## 3) Create a symbolic link to point to the WildFly installation directory

    $ ln -s /opt/wildfly-$Version_Number /opt/wildfly

## 4) Give access to WildFly group and user

    $ chown -RH wildfly: /opt/wildfly

## 5) Configure Wildfly to be run as a service
Create a directory where we will copy the wildfly.conf file. This file is a part of the WildFly package that you downloaded and installed.
Copy the wildfly.conf file from the package files to the newly created directory

    $ mkdir -p /etc/wildfly
    $ cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf /etc/wildfly/

copy the launch.sh script from the WildFly package to the /opt/wildfly/bin/ folder
 

    $ cp /opt/wildfly/docs/contrib/scripts/systemd/launch.sh /opt/wildfly/bin/

make the script executable through the following command

    $ sh -c 'chmod +x /opt/wildfly/bin/*.sh'

The last file to copy is the wildfly.service unit file to your system’s services folder /etc/systemd/system

    $ cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service /etc/systemd/system/

you have to inform your system that you have added a new unit file. This can be done by reloading the systemctl daemon
Start Service

    $ systemctl daemon-reload
    $ systemctl start wildfly

You can verify if all is working well by checking the status of the service as follows

    $ systemctl status wildfly

# 2. Configure WildFly

## Create a WildFly Administrator

    /opt/wildfly/bin/add-user.sh

## Open the Administrative console

    http://localhost:9990/console

## Managing the Administrative console remotely
Open the wildfly.conf file through the following command

    $ vim /etc/wildfly/wildfly.conf

Add the following lines to the end of the file

    # The address console to bind to
    WILDFLY_CONSOLE_BIND=0.0.0.0

Open the launch .sh script file through the following command

    sudo nano /opt/wildfly/bin/launch.sh

Change the highlighted lines to the following and restart service

    $WILDFLY_HOME/bin/domain.sh -c $2 -b $3
    else
    $WILDFLY_HOME/bin/standalone.sh -c $2 -b $3

    =>

    $WILDFLY_HOME/bin/domain.sh -c $2 -b $3 -bmanagement $4
    else
    $WILDFLY_HOME/bin/standalone.sh -c $2 -b $3 -bmanagement $4

    $ systemctl restart wildfly

edit the wildfly.service file through the following command

    ExecStart=/opt/wildfly/bin/launch.sh $WILDFLY_MODE $WILDFLY_CONFIG $WILDFLY_BIND
    =>
    ExecStart=/opt/wildfly/bin/launch.sh $WILDFLY_MODE $WILDFLY_CONFIG $WILDFLY_BIND $WILDFLY_CONSOLE_BIND

Since we have changed the service unit file, let us notify the system and restart service

    $ systemctl daemon-reload
    $ systemctl restart wildfly

# 3. Open the Administrative Console CLI

    $ cd /opt/wildfly/bin/
    $ ./jboss-cli.sh --connect
