7. Write a Debian based configuration file that will create a bridge device from eth0 and eth1.
-----------------------------------------------------------------------------------------------

Конфигурационния файл в Debian е /etc/network/interfaces.

Вариант 1, динамичен IP адрес:

	iface lo inet loopback

	iface eth0 inet manual
	iface eth1 inet manual
	
	iface br0 inet dhcp
		bridge_ports eth0 eth1

Вариант 2, статичен IP адрес:

	iface lo inet loopback

	iface eth0 inet manual
	iface eth1 inet manual
	
	iface br0 inet static
		address 10.1.1.200
		network 10.1.1.0
		gateway 10.1.1.254
		netmask 255.255.255.0
		broadcast 10.1.1.255
		bridge_ports eth0 eth1
