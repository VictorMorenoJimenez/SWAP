# Práctica 4. Asegurar la granja web.

**Objetivos** de la práctica 4:

- [ ]  Instalar un certificado SSL para configurar el acceso HTTPS a los servidores.
- [ ] Configurar las reglas del cortafuegos para proteger la granja web.
- [ ] Primera tarea opcional: Configurar nginx adecuadamente para aceptar y balancear correctamente tanto el tráfico HTTP como el tráfico HTTPS.
- [ ] Segunda tarea opcional. Realizar la instalación de un certificado del proyecto Certbot en lugar de uno autofirmado. Es necesario disponer de un dominio real con IP pública.

## Instalar certificado SSL auto firmado en máquina 1.
Para generar un certificado autofirmado en Ubuntu, procedemos a habilitar el módulo ssl. Con un simple comando podemos realizarlo:
```bash
  a2enmod ssl
```

A continuación creamos un directorio donde guardar los certificados que vamos a generar:

```bash
  mkdir /etc/apache2/ssl
```

Una vez creado el directorio procedemos a crear el certificado de apache junto con su llave tal y como se indica en el guión.

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout
/etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```

![Apache2 cert](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P4/img/apache2cert.png)

A continuación tenemos que activar la página que acabamos de configurar:

```bash
  a2ensite default-ssl
  systemctl restart apache2
```

Una vez reiniciado el servicio la máquina ya está preparada para aceptar tráfico https. Lo comprobamos lanzando una petición desde el host:

![peticionhttps](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P4/img/apachehttps.png)

## Opcional 1. Configurar certificados en ambas máquinas y configurar balanceador nginx para servir HTTP y  HTTPS.
Para configurar los certificados en la máquina 2 procedemos de misma manera que en la máquina 1 con la única diferencia que en este caso no debemos generar un certificado nuevo, utilizaremos el mismo certificado que en la máquina 1. Recordamos los pasos a seguir:

```bash
  a2enmod ssl
  mkdir /etc/apache2/ssl && cd /etc/apache2/ssl
```

Ahora debemos de copiar los certificados generados en la máquina 1. Para ellos utilizaremos el comando scp.
Una vez en la máquina 1.
```bash
  cd /etc/apache2/ssl
  scp apache.crt victorswap@192.168.56.51:/home/victorswap
  scp apache.key victorswap@192.168.56.51:/home/victorswap
```

Ahora en la máquina 2 movemos el certificado a la carpeta creada anteriormente.

```bash
  cd /home/victorswap
  mv apache.* /etc/apache2/ssl
```

### Configurar nginx para balancear tanto HTTP como HTTPS
Ahora, accedemos a la máquina 3, el balanceador y procedemos a editar el fichero de configuración de nginx. Debemos modificarlo para que acepte tanto el tráfico por el puerto 443 como el tráfico por el puerto 80. Con las siguientes directivas, por defecto nginx balanceará a https pero como tiene la directiva listen 80 también balanceará tráfico HTTP.

![nginxConf](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P4/img/nginxConf.png)

Nótese que en las directivas ssl_certificate y ssl_certificate_key debemos poner la ruta a la carpeta donde hayamos copiado los certificados creados en la máquina 1. Lo podemos hacer como en el paso anterior utilizando scp o bien utilizando rsync o incluso podemos copiar el certificado manualmente (no se recomienda). 

Si recapitulamos, hemos creado un certificado ssl en la máquina 1 para asegurar nuestra granja web. Éste certificado lo hemos copiado en la máquina 2 y en el balanceador nginx. Hemos configurado los ficheros de configuración de apache2 en las máquinas finales y el fichero de configuración de nginx en el balanceador para aceptar tráfico HTTP y HTTPS.

A continuación se muestra como al hacer peticiones al balanceador 192.168.56.52 redirige el tráfico entre la máquina 1 y la máquina 2.

![balanceDemo](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P4/img/demostracionBalance.png)

Si nos fijamos en los logs, sobre las 11.28 nginx ha ido balanceando entre m1 y m2 tanto HTTP como HTTPS



## Configurar reglas iptables cortafuegos con script.

Iptables es un software que actúa de cortafuegos y se permite al super usuario crear reglas para decidir hacía donde redirigir cierto tipo de paquetes. 

En nuestro caso debemos configurar el cortafuegos de una de las máquinas. En nuestro caso hemos elegido la máquina uno. Se ha creado un script script.sh con todas las directivas necesarias para habilitar todo el tráfico en localhost, y permitir accesos al puerto 80 y puerto 443. A continuación se muestraa el contenido del script.

``` bash
#!/bin/bash

# (1) Eliminar todas las reglas (configuración limpia)
iptables -F
iptables -X
iptables -Z
iptables -t nat -F

# (2) Política por defecto: denegar todo el tráfico
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

# (3) Permitir cualquier acceso desde localhost (interface lo)
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# (4) Abrir el puerto 22 para permitir el acceso por SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

# (5) Abrir los puertos HTTP (80) de servidor web
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

# (6) Abrir el puerto HTTPS (443) de servidor web
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT
```

Para hacer que el script se arranque al inicio de el sistema, hemos creado un servicio en systemd. Este servicio se ejecutará al inicio del sistema justo depués de se active el servicio network y permanecerá inactivo hasta el próximo inicio de sistema. Para crear un servicio del sistema debemos añadir un fichero .service como se muestra a continuación.

```bash
  sudo su
  cd /etc/systemd/system/
  nano iptables.service
```

Configuramos el servicio para que arranque después del servicio network y que ejecute el script creado anteriormente:

```bash
  [Unit]
  Description=Ip tables service to secure the host
  After=network.target
  StartLimitIntervalSec=0

  [Service]
  Type=simple
  Restart=always
  RestartSec=1
  User=root
  ExecStart=/home/victorswap/script.sh

  [Install]
  WantedBy=multi-user.target
```

Una vez creado el servicio hacemos que se ejecute al inicio del sistema:

```bash
  systemctl enable iptables
```

Reiniciamos el sistema y comprobamos que ha creado las reglas adecuadamente:


![balanceDemo](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P4/img/iptables.png)

### Crear servicio systemd para la creación de las reglas iptables.

## Opcional 2. Instalación certificado Certbot.
