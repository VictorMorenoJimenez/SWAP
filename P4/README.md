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
  sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

El fichero de configuración deber quedar de la siguiente manera:

```bash
  [mysqld_safe]
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql

skip-external-locking
key_buffer_size		= 16M
max_allowed_packet	= 16M
thread_stack		= 192K
thread_cache_size       = 8
query_cache_limit	= 1M
query_cache_size        = 16M

log_error = /var/log/mysql/error.log

server-id		= 1
log_bin			= /var/log/mysql/mysql-bin.log

expire_logs_days	= 10

max_binlog_size   = 100M

```

La configuración del esclavo en la máquina 2, si tenemos versiones actualmes de MySQL, es exactamente igual que la de M1, únicamente cambia el server-id, que en este caso será 2. 

```bash
  [mysqld_safe]
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql

skip-external-locking
key_buffer_size		= 16M
max_allowed_packet	= 16M
thread_stack		= 192K
thread_cache_size       = 8
query_cache_limit	= 1M
query_cache_size        = 16M

log_error = /var/log/mysql/error.log

server-id		= 2
log_bin			= /var/log/mysql/mysql-bin.log

expire_logs_days	= 10

max_binlog_size   = 100M

```

Reiniciamos el servicio y procedemos a acceder a la máquina 1. Ahora, tenemos que crear un usuario en la base de datos con permisos para que pueda replicar la base de datos.

```bash
  mysql -uroot -p
  mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
  Query OK, 0 rows affected (0.01 sec)

  mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
  Query OK, 0 rows affected, 1 warning (0.02 sec)

  mysql> FLUSH PRIVILEGES;
  Query OK, 0 rows affected (0.01 sec)

  mysql> FLUSH TABLES;
  Query OK, 0 rows affected (0.02 sec)

  mysql> FLUSH TABLES WITH READ LOCK;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> SHOW MASTER STATUS;
  +------------------+----------+--------------+------------------+-------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +------------------+----------+--------------+------------------+-------------------+
  | mysql-bin.000001 |      980 |              |                  |                   |
  +------------------+----------+--------------+------------------+-------------------+
  1 row in set (0.00 sec)


```

Muy importante, guardar la información que nos muestra el comando show master status. Ya que a continuación la introduciremos en la máquina esclava para poder realizar la replicación.

