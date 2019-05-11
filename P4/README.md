# Práctica 5. Replicación de bases de datos MySQL.

**Objetivos** de la práctica :

- [ ] Copiar archivos de copia de segurida mediante ssh.
- [ ] Clonar manualmente BD entre máquinas.
- [ ] Configura la estructura maestro-esclavo entre dos máquinas para realizar el clonado automático de la información.


## Generar copia de seguridad mediante ssh.
Para generar una copia de seguridad de base de datos de una máquina M1 a otra máquina M2, lo podemos hacer manualmente utilizando el comando mysqldump. Es muy sencillo, en la máquina 1 procedemos a utilizar el comando mysqldump en la base de datos que queremos replicar. Suponiendo que nuestra base de datos se llama contactos.

```bash
  mysqldump -u root -p contactos > contactosDB.sql
```


Éste fichero será el que utilizemos en la máquina 2 donde queremos hacer la copia de seguridad para importarlo. Una vez en la máquina 2, para copiar el fichero:

```bash
  scp 192.168.56.50:/home/victorswap/contactos.sql /tmp/
  cd /tmp
  mysql -u root -p contactos < contactosDB.sql
```


Antes de importar el fichero, hay que crear la base de datos ya que el fichero .sql de importación lo único que hace es rellenar la base de datos, no la crea. Por eso debemos crear la base de datos antes de ejecutar el comando de importación anterior.

```bash
  CREATE DATABASE contactos;
```

Con esto ya tenemos hecha la copia de seguridad, el único inconveniente es que ésto se tiene que realizar a mano y si tienes muchas máquinas es inviable. Para solucionar ésto vamos a crear una configuración maestro-esclavo para que las copias de seguridad se realicen periódicamente de forma automatizada.


### Configurar maestro-esclavo, clonado automático.
Para empezar accedemos a la máquina 1 que será nuestro Maestro y editamos el fichero de configuración como root. 

```bash
  sudo nano /etc/mysql/my.cnf
```

El fichero de configuración deber quedar de la siguiente manera:

```bash
  sudo nano /etc/mysql/my.cnf
```

