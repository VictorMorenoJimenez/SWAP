# Práctica 5. Replicación de bases de datos MySQL.

**Objetivos** de la práctica :

- [ ] Copiar archivos de copia de segurida mediante ssh.
- [ ] Clonar manualmente BD entre máquinas.
- [ ] Configura la estructura maestro-esclavo entre dos máquinas para realizar el clonado automático de la información.


## Generar copia de seguridad mediante ssh.
Para generar un certificado autofirmado en Ubuntu, procedemos a habilitar el módulo ssl. Con un simple comando podemos realizarlo:
```bash
  a2enmod ssl
```

![peticionhttps](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P5/img/)

## Clonar manualmente BD entre máquinas.
Para configurar los certificados en la máquina 2 procedemos de misma manera que en la máquina 1 con la única diferencia que en este caso no debemos generar un certificado nuevo, utilizaremos el mismo certificado que en la máquina 1. Recordamos los pasos a seguir:

```bash
  a2enmod ssl
  mkdir /etc/apache2/ssl && cd /etc/apache2/ssl
```

### Configurar maestro-esclavo, clonado automático.
Ahora, accedemos a la máquina 3, el balanceador y procedemos a editar el fichero de configuración de nginx. Debemos modificarlo para que acepte tanto el tráfico por el puerto 443 como el tráfico por el puerto 80. Con las siguientes directivas, por defecto nginx balanceará a https pero como tiene la directiva listen 80 también balanceará tráfico HTTP.

![nginxConf](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P4/img/nginxConf.png)

Nótese que en las directivas ssl_certificate y ssl_certificate_key debemos poner la ruta a la carpeta donde hayamos copiado los 



