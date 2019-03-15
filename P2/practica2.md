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
## Prueba funcionamiento copia de archivos por ssh.

A continuación, utilizaremos la herramienta **rsync** junto con **ssh** para copiar una **carpeta remota** desde la máquina 2 hasta la máquina 1. Para ello utilizaremos el siguiente comando:
```bash
   rsync -avz -e ssh ipMáquina1:$RutaACarpetaM1 $RutaCarpetaM2
```

De éste modo la carpeta con ruta __ipMáquina1:$RutaACarpetaM1__ se sincronizará con la carpeta con ruta __RutaCarpetaM2__.

Vemos una foto con un ejemplo real:

![uso curl](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P2/images/pruebarsynccopiarcarpeta.png)
