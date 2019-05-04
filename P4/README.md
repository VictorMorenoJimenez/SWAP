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

Si recapitulamos, hemos creado un certificado ssl en la máquina uno para asegurar nuestra granja web.


## Configurar reglas iptables cortafuegos con script.

### Crear servicio systemd para la creación de las reglas iptables.

## Opcional 2. Instalación certificado Certbot.
