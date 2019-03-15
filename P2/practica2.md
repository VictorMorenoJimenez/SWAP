# Práctica 2. Clonar la información de un sitio web.

**Objetivos** de la práctica 2:

- [ ] Probar funcionamiento de la copia de archivos por **ssh**
- [ ] Clonar contenido entre máquinas.
- [X] Configurar el **ssh** para acceder a máquinas remotas **sin contraseña**.
- [ ] Establecer una tarea en **cron** que se ejecute cada hora para mantener actualizado el contenido de **/var/www/**
      entre las dos máquinas.
      
Como se puede observar hay una tarea que ya está completada y es que se configuró en la práctica 1. Recordamos que para poder acceder
sin contraseña a una máquina hay que primero generar claves y segundo hacer que las máquinas se conozcan. Recordamos los dos comandos:

Para generar claves, privada y pública:

```bash
   ssh-keygen
```

Para poder acceder sin contraseña:
```bash
   ssh-copy-id usuario@ipMáquina
```

![ssh-copy-id](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P2/images/sshcopyIDm1tom2.png)

Observamos que nos aparece un mensaje de que ya tenemos las claves copiadas y es que, ésto se realizó en la **práctica1**.


## Prueba funcionamiento copia de archivos por ssh

A continuación, utilizaremos la herramienta **rsync** junto con **ssh** para copiar una **carpeta remota** desde la máquina 2 hasta la máquina 1. Para ello utilizaremos el siguiente comando:
```bash
   rsync -avz -e ssh ipMáquina1:$RutaACarpetaM1 $RutaCarpetaM2
```

De éste modo la carpeta con ruta __ipMáquina1:$RutaACarpetaM1__ se sincronizará con la carpeta con ruta __RutaCarpetaM2__.

Vemos una foto con un ejemplo real:

![pruebaRsync](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P2/images/pruebarsynccopiacarpeta.png)

Una vez probado rsync únicamente nos queda crear una tarea cron que sincronice los dos servidores web.


## Tarea cron
La finalidad de ésta tarea cron será mantener el servidor web de **M2** actualizado con **M1**. Ésto se va a llevar a cabo
utilizando la herramienta **cron**. Cron es un servicio de planificación de tareas basado en tiempo muy itilizado en sistemas operativos **Unix**. "Cron" es una abreviatura de _cronograph_.
Para llevar a cabo nuestro objetivo simplemente tenemos que añadir una línea nueva al fichero _/etc/crontab_. En éste fichero se recogen las tareas que se ejecutan periódicamente por cron. Abrimos _/etc/crontab_ con nuestro editor preferidoy insertamos al final la siguiente linea:

```bash
   00 * * * * victorswap      rsync -avz -e ssh 192.168.56.50:/var/www/ /var/www
```

Hay que tener en cuenta que, como se quiere copiar a la **M2** la carpeta desde **M1** la ip que debemos poner en el comando
es la de **M1**.

De éste modo, cada vez que los minutos marquen el 00, todas las horas, todos los días y todos los meses se ejecutará la tarea.
En definitiva cada hora de todo el año.

![crontab](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P2/images/crontabm2copiaam1.png)
