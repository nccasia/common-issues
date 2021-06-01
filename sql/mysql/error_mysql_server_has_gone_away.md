<b>Issue</b>: HOW DO I FIX THE ERROR "MYSQL SERVER HAS GONE AWAY"? \
<b>Solution</b>:
    The MySQL server has gone away (error 2006) has two main causes and solutions:

    Server timed out and closed the connection. To fix, check that wait_timeout mysql variable in your my.cnf configuration file is large enough, eg wait_timeout = 28800
    You may also need to increase the innodb_log_file_size mysql variable in your my.cnf configuration to for example innodb_log_file_size = 128MB or higher.

    Server dropped an incorrect or too large packet. If mysqld gets a packet that is too large or incorrect, it assumes that something has gone wrong with the client and closes the connection. To fix, you can increase the maximal packet size limit max_allowed_packet in my.cnf file, eg. set max_allowed_packet = 128M, then restart your MySQL server: sudo /etc/init.d/mysql restart
    
    Once you’ve made these changes, and restarted your MySQL or MariaDB server, the issue should be fixed and no more error triggered. If not, try increase the 128M values to 256M for example. In websites with a lot of traffic and data, you might even need to increase the value to 1024M.