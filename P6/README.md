# Práctica 6. Servidor de disco NFS.

**Objetivos** de la práctica :

- [ ] Configurar una máquina como servidor de disco NFS y exportar una carpeta a los clientes.
- [ ] Montar en dos máquinas cliente la carpeta exportada por el servidor.
- [ ] Comprobar que la información que se escribe en una máquina en dicha carpeta, se actualiza en las otras dos.


## Instalar y configurar servidor NFS.
Para empezar, instalamos los paquetes necesarios nfs.

```bash
   sudo apt-get install nfs-kernel-server nfs-common rpcbind
```

Ahora creamos una carpeta que será la carpeta a compartir con nuestros clientes. Ésta carpeta tiene que tener todos los permisos y no tiene que tener dueño. Para ello ejecutamos los siguientes comandos:

```bash
   mkdir /dat/compartida
   sudo chown nobody:nogroup /dat/compartida/
   sudo chmod -R 777 /dat/compartida/
```

También debemos modificar la configuración de NFS, del fichero /etc/exports. Al final del fichero añadimos una línea con los clientes que vvan a compartir la carpeta en cuestión.

```bash
root@ubuntu:/home/victor# cat /etc/exports 
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#

/dat/compartida/	192.168.56.50(rw)	192.168.56.51(rw)

```
## Configurar dos clientes 

Una vez configurado el servidor procedemos con los dos clientes. La configuración para ambos es la misma con lo cual, únicamente mostraremos la configuración realizada para la máquina 1.


Primero debemos instalar los siguientes paquete:

```bash
  sudo apt-get install nfs-common rpcbind
```

Igual que en el servidor, craemos la carpeta y la configuramos igual. Estando en el home del usuario:

```bash
  mkdir carpetacliente
  chmod -R 777 carpetacliente/
```

Una vez creada, tenemos que montar la carpeta remota del servidor en la máquina 1. Para ello usamos la orden mount.

```bash
  sudo mount 192.168.56.200:/dat/compartida carpetacliente/
```

Esta configuración se ha de replicar en la máquina 2.


### Montar carpeta al inicio del sistema
Para crear la carpeta al inicio del sistema simplemente añadimos la siguiente línea en el fichero /etc/fstab. Tanto en la máquina1 como en la máquina2.

```bash
  victorswap@swapM1:~/carpetacliente$ cat /etc/fstab 
  # /etc/fstab: static file system information.
  #
  # Use 'blkid' to print the universally unique identifier for a
  # device; this may be used with UUID= as a more robust way to name devices
  # that works even if disks are added and removed. See fstab(5).
  #
  # <file system> <mount point>   <type>  <options>       <dump>  <pass>
  /dev/mapper/swapM1--vg-root /               ext4    errors=remount-ro 0       1
  # /boot was on /dev/sda1 during installation
  UUID=c8d622d8-27c0-486b-878a-823ae9165bf5 /boot           ext2    defaults        0       2
  /dev/mapper/swapM1--vg-swap_1 none            swap    sw              0       0

  192.168.56.200:/dat/compartida /home/victorswap/carpetacliente/	nfs	auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800	0	0
```


## Demostración

