#Práctica 6. Discos en RAID#

##Los objetivos concretos de esta práctica son:##
Configurar dos discos en RAID 1. Los discos se añadirán a un sistema ya instalado y funcionando, de forma que en total tendremos tres discos (el sistema operativo estará ya instalado en /dev/sda y añadiremos dos discos más, que serán el /dev/sdb y el /dev/sdc, para configurar el dispositivo de almacenamiento RAID en estos dos discos nuevos de igual tamaño).
Hacer pruebas de retirar y añadir un disco “en caliente”, y comprobar que el RAID sigue funcionando correctamente. 


##Configuración del RAID##
Antes de configurar debemos añadir dos discos duros estando la máquina apagada, tras esto instalamos mdadm y montamos el RAID.
sudo apt-get install mdadm
sudo mdadm -C /dev/md/mdo --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc
sudo mkdir /datos
sudo mount /dev/md/mdo /datos
Siendo /dev/sdb y /dev/sdc los dos nuevos discos.
Para comprobar el estado del RAID ejecutamos
sudo mdadm --detail /dev/md0

![img](https://github.com/jomoca/swap1516/blob/master/practica5/imagenes/estado_raid.png)

Para finalizar el proceso, conviene configurar el sistema para que monte el dispositivo
RAID creado al arrancar el sistema. Para ello debemos editar el archivo /etc/fstab y
añadir la línea correspondiente para montar automáticamente dicho dispositivo.
Conviene utilizar el identificador único de cada dispositivo de almacenamiento en lugar
de simplemente el nombre del dispositivo (aunque ambas opciones son válidas). Para
obtener los UUID de todos los dispositivos de almacenamiento que tenemos, debemos
ejecutar la orden:
ls -l /dev/disk/by-uuid/

Anotaremos el correspondiente al dispositivo RAID que hemos creado. Ahora ya
podemos añadir al final del archivo /etc/fstab la línea para que monte automáticamente
el dispositivo RAID, que será similar a:
UUID=ccbbbbcc-dddd-eeee-ffff-aaabbbcccddd /dat ext2 defaults 0 0

![img](https://github.com/jomoca/swap1516/blob/master/practica5/imagenes/ss.png)
