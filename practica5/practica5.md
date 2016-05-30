#Práctica 5. Replicación de bases de datos MySQL#


Los objetivos concretos de esta práctica son:
	Copiar archivos de copia de seguridad mediante ssh.
	Clonar manualmente BD entre máquinas.
	Configurar la estructura maestro-esclavo entre dos máquinas para realizar el
	clonado automático de la información.

##**Crear una BD e insertar datos**##

Para el resto de la práctica debemos crearnos una BD en MySQL e insertar algunos
datos. Así tendremos datos con los cuales hacer las copias de seguridad. En todo
momento usaremos la interfaz de línea de comandos del MySQL:

mysql -uroot -p


###creamos la base de datos:###

mysql> create database CONTACTOS;


###creamos tablas:###

mysql> create table datos (nombre varchar (100), tlf int);

###Insertamos datos en las tablas:###

mysql> insert into datos (nombre, tlf) values ("pepe",95834987);

![img](https://github.com/jomoca/swap1516/blob/master/practica5/imagenes/basedatos.png)

###Replicar una BD MySQL con mysqldump###

La sintaxis de uso es:
mysqldump CONTACTOS -u root -p > /root/ CONTACTOS.sql

Esto puede ser suficiente, pero tenemos que tener en cuenta que los datos pueden estar actualizándose constantemente en el servidor de BD principal. En este caso, antes de hacer la copia de seguridad en el archivo .SQL debemos evitar que se acceda al BD para cambiar nada. Así, en el servidor de BD principal (maquina1) hacemos:
mysql -u root –p
mysql> FLUSH TABLES WITH READ LOCK;
mysql> quit

Ahora ya sí podemos hacer el mysqldump para guardar los datos. En el servidor principal (maquina1) hacemos:
mysqldump CONTACTOS -u root -p > /root/CONTACTOS.sql

Como habíamos bloqueado las tablas, debemos desbloquearlas (quitar el “LOCK”):
mysql -u root –p
mysql> UNLOCK TABLES;
mysql> quit

Ya podemos ir a la máquina esclavo (maquina2, secundaria) para copiar el archivo .SQL con todos los datos salvados desde la máquina principal (maquina1):
scp 172.16.101.128/root/CONTACTOS.sql /root/

![img](https://github.com/jomoca/swap1516/blob/master/practica5/imagenes/copiar base de datos.png)

Con el archivo de copia de seguridad en el esclavo ya podemos importar la BD completa en el MySQL. Para ello, en un primer paso creamos la BD:
mysql -u root –p
mysql> CREATE DATABASE ‘CONTACTOS’;
mysql> quit

Y en un segundo paso restauramos los datos contenidos en la BD (se crearán las tablas en el proceso):
mysql -u root -p CONTACTOS < /root/CONTACTOS.sql

Replicación de BD mediante una configuración maestro-esclavo Partimos teniendo clonadas las bases de datos en ambas máquinas. Lo primero que debemos hacer es la configuración de mysql del maestro. Para ello editamos, como root, el /etc/mysql/my.cnf para realizar las modificaciones que se describen a continuación.
Comentamos el parámetro bind-address que sirve para que escuche a un servidor:
#bind-address 127.0.0.1

Le indicamos el archivo donde almacenar el log de errores. De esta forma, si por ejemplo al reiniciar el servicio cometemos algún error en el archivo de configuración, en el archivo de log nos mostrará con detalle lo sucedido:
log_error = /var/log/mysql/error.log

Establecemos el identificador del servidor.
server-id = 1

El registro binario contiene toda la información que está disponible en el registro de actualizaciones, en un formato más eficiente y de una manera que es segura para las transacciones:
log_bin = /var/log/mysql/bin.log

Guardamos el documento y reiniciamos el servicio:
/etc/init.d/mysql restart

Si no nos ha dado ningún error la configuración del maestro, podemos pasar a hacer la configuración del mysql del esclavo (vamos a editar como root el /etc/mysql/my.cnf). La configuración es similar a la del maestro, con la diferencia de que el server-id en esta ocasión será 2. 

Reiniciamos el servicio en el esclavo:
/etc/init.d/mysql restart

y una vez más, si no da ningún error, habremos tenido éxito. Podemos volver al maestro para crear un usuario y darle permisos de acceso para la replicación. Entramos en mysql y ejecutamos las siguientes sentencias:
mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%'
IDENTIFIED BY 'esclavo';
mysql> FLUSH PRIVILEGES;
mysql> FLUSH TABLES;
mysql> FLUSH TABLES WITH READ LOCK;

Para finalizar con la configuración en el maestro, obtenemos los datos de la base de datos que vamos a replicar para posteriormente usarlos en la configuración del esclavo:
mysql> SHOW MASTER STATUS;

![img](https://github.com/jomoca/swap1516/blob/master/practica5/imagenes/show_master.png)


Volvemos a la máquina esclava, entramos en mysql y le damos los datos del maestro. Como indicábamos antes, estos datos se pueden introducir directamente en el archivo de configuración si trabajamos con versiones inferiores a mysql 5.5. Si no es así, en el entorno de mysql ejecutamos la siguiente sentencia (ojo con la IP, "master_log_file" y del "master_log_pos" del maestro):
mysql> CHANGE MASTER TO MASTER_HOST='172.16.101.128',MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='bin.000004', MASTER_LOG_POS=107, MASTER_PORT=3306;

![img](https://github.com/jomoca/swap1516/blob/master/practica5/imagenes/ll.png)

Por último, arrancamos el esclavo y ya está todo listo para que los demonios de MySQL de las dos máquinas repliquen automáticamente los datos que se introduzcan/modifiquen/borren en el servidor maestro:
mysql> START SLAVE;

Comprobamo que todo funciona bien.

![img](https://github.com/jomoca/swap1516/blob/master/practica5/imagenes/show_slave.png)
