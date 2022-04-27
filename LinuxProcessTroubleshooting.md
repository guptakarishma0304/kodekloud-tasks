Question:
The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.

Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not hosted any code yet on these servers so you need not to worry about if Apache isn't serving any pages or not, just make sure service is up and running. Also, make sure Apache is running on port 8087 on all app servers.

Solution:

# try checking status of apache services
[root@stapp01 tony]# systemctl status apache-services
Unit apache-services.service could not be found.
[root@stapp01 tony]# systemctl status apache-httpd
Unit apache-httpd.service could not be found.
[root@stapp01 tony]# systemctl status apache
Unit apache.service could not be found.

# try checking status of apache services httpd
[root@stapp01 tony]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
# Restart httpd service
[root@stapp01 tony]# sudo systemctl start httpd
Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.

# check the config file to check the port mentioned
[root@stapp01 tony]# sudo vi /etc/httpd/conf/httpd.conf


# run netstat command to check other system running on same post 8087
[root@stapp01 tony]# sudo netstat -anp | grep LISTEN
tcp        0      0 127.0.0.1:8087          0.0.0.0:*               LISTEN      612/sendmail: accep 
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/init              
tcp        0      0 127.0.0.11:46735        0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      513/sshd            
tcp6       0      0 :::111                  :::*                    LISTEN      498/rpcbind         
tcp6       0      0 :::22                   :::*                    LISTEN      513/sshd            
unix  2      [ ACC ]     STREAM     LISTENING     5772494  1/init               /run/systemd/private
unix  2      [ ACC ]     SEQPACKET  LISTENING     5757595  1/init               /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     5757596  1/init               /run/systemd/journal/stdout
unix  2      [ ACC ]     STREAM     LISTENING     5768048  1/init               /run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     5768049  1/init               /var/run/rpcbind.sock
unix  2      [ ACC ]     STREAM     LISTENING     5784933  502/gssproxy         /run/gssproxy.sock
unix  2      [ ACC ]     STREAM     LISTENING     5784930  502/gssproxy         /var/lib/gssproxy/default.sock

# grep on PORT number
[root@stapp01 tony]# sudo netstat -anp | grep 8087
tcp        0      0 127.0.0.1:8087          0.0.0.0:*               LISTEN      612/sendmail: accep 

# kill the process using PID number "612"
[root@stapp01 tony]# sudo kill -9 612

# grep again to check whether the process is killed or not
[root@stapp01 tony]# sudo netstat -anp | grep 8087

# Restart the service
[root@stapp01 tony]# systemctl restart httpd

# check the status of service
[root@stapp01 tony]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-04-27 09:07:42 UTC; 11s ago
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 701 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=1/FAILURE)
 Main PID: 1018 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /docker/8b45cc342efdf5802b75b32770e4c3897c5ed23167a42cb515c344c6d9a020f3/system.slice/httpd.service
           ├─1018 /usr/sbin/httpd -DFOREGROUND
           ├─1020 /usr/sbin/httpd -DFOREGROUND
           ├─1021 /usr/sbin/httpd -DFOREGROUND
           ├─1022 /usr/sbin/httpd -DFOREGROUND
           ├─1023 /usr/sbin/httpd -DFOREGROUND
           └─1024 /usr/sbin/httpd -DFOREGROUND

Apr 27 09:07:42 stapp01.stratos.xfusioncorp.com systemd[1]: Starting The Apache HTTP Server...
Apr 27 09:07:42 stapp01.stratos.xfusioncorp.com httpd[1018]: AH00558: httpd: Could not reliably determine the server's fully quali...ssage
Apr 27 09:07:42 stapp01.stratos.xfusioncorp.com systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.

# login to other server and check the status of httpd service and then check the config file to confirm the port number

cat /etc/httpd/conf/httpd.conf | grep Listen