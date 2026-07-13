1. local host == 127.0.0.1
2. apache web sever default wants 80 server
	1. if you want to change the port
				``` sudo nano /etc/apache2/port.conf

			change Listen 80 to Listen<port number ```
3. curl localhost:<<port number>>
   4.  