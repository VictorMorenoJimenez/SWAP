# Práctica 1. Preparación de las herramientas.

**Objetivos** de la práctica 1:

- [ ] Instalar ubuntu server con **LAMP** y **curl**
- [ ] Instalar y configurar **ssh** para acceder entre máquinas.
- [ ] Acceder mediante **curl** de una máquina a la otra.
- [ ] Acceder mediante **ssh** de una máquina a la otra.

Para la realización de esta primera práctica, hemos instalado **ubuntu server 16.04** en dos máquinas virtuales utilizando **virtualbox**.
Utilizando la configuración por defecto e instalando el stack **LAMP** en ambas máquinas. Podemos utilizar la interfaz de instalación o
una vez instaladas las máquinas ejecutar el siguiente comando:

```bash
  sudo apt-get install apache2 mysql-server mysql-client
```

Una vez instalado **LAMP**, instalamos con el gestor de paquetes la herramienta curl para transferencia de archivos. Ejecutamos en terminal:

```bash
  sudo apt-get install curl
```

También, si no hemos instalado **ss**h con la interfaz de instalación lo instalamos con el siguiente comando:

```bash
  sudo apt-get openssh-server
```

Para ver si tenemos el servicio activo:

```bash
  sudo systemctl status ssh
```

En el caso de no tenerlo activo debemos hacer que se inicie con el sistema y le hacemos start.

```bash
  sudo systemctl enable ssh
  sudo systemctl start ssh
```

Una vez llegados a este punto, nuestra lista de objetivos es la siguiente:

- [X] Instalar ubuntu server con **LAMP** y **curl**
- [X] Instalar y configurar **ssh**.
- [ ] Acceder mediante **curl** de una máquina a la otra.
- [ ] Acceder mediante **ssh** de una máquina a la otra.

Para poder conectar ambas máquinas configuramos una interfaz de red en **virtual box** _host only_ y configuramos el siguiente fichero de configuración
en las máquinas:
```bash
  sudo nano /etc/network/interfaces
```

Una configuración estándart de una máquina para darle una ip sería la siguiente:
```bash
  source /etc/network/interfaces.d/*

  # The loopback network interface
  auto lo
  iface lo inet loopback

  # The primary network interface
  auto enp0s3
  iface enp0s3 inet dhcp

  auto enp0s8
  iface enp0s8 inet static
  address 192.168.56.50
  netmask 255.255.255.0

```
Dónde **enp0s3** es la interfaz que hace **NAT** y **enp0s8** es nuestro **vboxnet0** el adaptador _host only_ .

En este momento ya tenemos todo listo para realizar las comprobaciones con **ssh** y **curl**. A continuación se muestran unas imágenes utilizando
dichas herramientas.
Si no queremos que ssh nos pida contraseña de una máquina a otra haremos lo siguiente: 
Primero generaremos las claves públicas y privadas que se utilizarán para que las máquinas se conozcan.

```bash
  ssh-keygen
```

Y ahora, si queremos por ejemplo que M1 se conecte directamente a M2:

```bash
  ssh-copy-id usuarioM2@IPM2
```

Y así es como podrems acceder directamente sin contraseña de M1 a M2.


## Pruebaa **curl**





