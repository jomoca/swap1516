#Práctica 3. Balanceo de carga#

En esta práctica configuraremos una red entre varias máquinas de forma que tengamos un balanceador que reparta la carga entre varios servidores finales.

Para ello intalaremos otro ubuntu server que funcione de balanceador.

Vamos a utilizar nginx y haproxy.

**##Instalación de nginx##**

Para instalar nginx en Ubuntu Server debemos realizar lo siguiente:

cd /tmp/
wget http://nginx.org/keys/nginx_signing.key
apt-key add /tmp/nginx_signing.key
rm -f /tmp/nginx_signing.key
Con esto añadimos la clave del repositorio de software.

Para añadir el repositorio debemos modificar el fichero /etc/apt/sources.list así:

echo "deb http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list
echo "deb-src http://nginx.org/packages/ubuntu/ lucid nginx" >> /etc/apt/sources.list

Ahora ya podemos instalar el paquete del nginx:

apt-get update
apt-get install nginx
Balanceo de carga

Para balancear la carga debemos editar el archivo /etc/nginx/conf.d/default.conf de la siguiente manera:

	upstream apaches {
		server 172.16.101.128;
		server 172.16.101.129;
	}
	server{
		listen 80;
		server_name balanceador;
		access_log /var/log/nginx/balanceador.access.log;
		error_log /var/log/nginx/balanceador.error.log;
		root /var/www/;
		location /
		{
			proxy_pass http://apaches;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_http_version 1.1;
			proxy_set_header Connection "";
		}
	}

Tras esto, reiniciamos nginx:
	service nginx restart

y comprobamos que funciona el balanceo usando curl http://172.16.101.130

![img](https://github.com/jomoca/swap1516/blob/master/practica3/imagenes/nginx_ubuntu1.png)
![img](https://github.com/jomoca/swap1516/blob/master/practica3/imagenes/nginx_ubuntu2.png)


Para usar el algoritmo de balanceo basado en prioridades basta con editar lo siguiente:

upstream apaches {
server 172.16.101.128 weight=1;
server 172.16.101.129 weight=2;
}
Siendo weight la prioridad, en este caso le hemos dado mas peso a la máquina ubuntu2


##**haproxy**##

Instalación de haproxy, para instalarlo basta con usar: sudo apt-get install haproxy

Configuración básica de haproxy como balanceador

Editamos su fichero de configuración, /etc/haproxy/haproxy.cfg, de la siguiente forma:

Archivo configuración haproxy

	global
		daemon
		maxconn 256
	defaults
		mode http
		contimeout	4000
		clitimeout	42000	
		srvtimeout	43000
	frontend http-in
		bind *:80
		default_backend servers
	backend servers
		server m1 172.16.101.128:80 maxconn 32
		server m2 172.16.101.129:80 maxconn 32

Ahora comprobamos que funciona correctamente, de la misma forma que nginx

![img](https://github.com/jomoca/swap1516/blob/master/practica3/imagenes/haproxy_ubuntu1.png)
![img](https://github.com/jomoca/swap1516/blob/master/practica3/imagenes/haproxy_ubuntu2.png)


Para utilizar un algoritmo por prioridad simplemente hay que editar lo siguiente del archivo de configuración

backend servers
    server m1 172.16.101.128:80 weight 1 maxconn 32
    server m2 172.16.101.129:80 weight 2 maxconn 32

Al igual que en nginx, le hemos dado mayor prioridad a la 172.16.101.129 respecto de 172.16.101.129

