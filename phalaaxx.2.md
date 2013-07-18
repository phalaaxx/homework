8. Create a job definition file for Apache web server for UpStart.
------------------------------------------------------------------

Създаваме файл /etc/init/apache2.conf със следното съдържание (Debian Wheezy, upstart 1.6.1):

	# apache2 upstart job
	#
	# This service maintains an apache2 http web server

	description     "Apache2 Web Server - Upstart job for Debian Wheezy"
	author          "Bozhin Zafirov <bozhin@abv.bg>"

	start on (runlevel [2345]
		and net-device-up
		and local-filesystems)
	stop on starting rc RUNLEVEL=[016]
	emits httpd

	# monitor service 
	respawn
	respawn limit 2 5

	kill timeout 60

	# variables
	env APACHE_RUN_USER=www-data
	env APACHE_RUN_GROUP=www-data
	env APACHE_RUN_DIR=/var/run/apache2
	env APACHE_LOG_DIR=/var/log/apache2
	env APACHE_LOCK_DIR=/var/lock/apache2
	env APACHE_PID_FILE=/var/run/apache2/apache2.pid

	# pre-start, similar to start in apache2ctl 
	pre-start script
	        mkdir -p ${APACHE_RUN_DIR}
	        mkdir -p ${APACHE_LOCK_DIR}
	        chown ${APACHE_RUN_USER} ${APACHE_LOCK_DIR}
	        rm -f ${APACHE_RUN_DIR}/*ssl_scache*
	end script

	# run apache2 daemon
	expect fork
	exec /usr/sbin/apache2 -k start



Следното е вариант за CentOS 6.4 (upstart 0.6.5):

	# apache2 upstart job
	#
	# This service maintains an apache2 http web server
	
	description     "Apache2 Web Server - CentOS 6.4"
	author          "Bozhin Zafirov <bozhin@abv.bg>"
	
	start on runlevel [2345]
	stop on starting rc RUNLEVEL=[016]
	emits httpd
	
	# monitor service 
	respawn
	respawn limit 2 5
	
	kill timeout 60
	
	# pre-start, similar to start in apache2ctl
	pre-start script
		mkdir -p /var/run/httpd
	end script

	# run apache2 daemon
	expect fork
	exec /usr/sbin/httpd -k start
