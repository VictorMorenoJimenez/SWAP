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
