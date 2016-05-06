#Práctica 2: Clonar la información de un sitio web#

Los objetivos concretos de esta segunda práctica son:
• aprender a copiar archivos mediante ssh
• clonar contenido entre máquinas
• configurar el ssh para acceder a máquinas remotas sin contraseña
• establecer tareas en cron

Disponemos de dos maquinas, ubuntu1 y ubuntu2

En la maquina ubuntu2 se guardará la información del directorio /var/www/ de la maquina ubuntu1.

**Para ello hay que permitir el acceso por ssh sin contraseña.**

En la maquina ubuntu2 haremos lo siguiente:

ssh-keygen -t dsa

Tras esto, debemos copiar las claves a la maquina ubuntu1 usando

ssh-copy-id -i .ssh/id_dsa.pub 172.16.101.128

Con esto conseguimos conectarnos mediante ssh a la maquina ubuntu1 sin contraseña


**Programar tareas con crontab**

cron es un administrador procesos en segundo plano que ejecuta procesos en el
instante indicado en el fichero crontab.
