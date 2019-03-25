# Práctica 3. Balanceo de carga.

**Objetivos** de la práctica 3:

- [ ] Configurar una máquina e instalarle el **nginx** como balanceador de carga.
- [ ] Configurar una máquina e instalarle el **haproxy** como balanceador de carga.
- [ ] Comprobar funcioamiento algoritmos e balanceo **round-robin** _Peso M1 = 2*Peso M2_
- [ ] Someter a la granja web a una alta carga, generada con herramienta **Apache Benchmark**, probar primero con nginx y después con haproxy.
- [ ] Comparativa en tiempos de ambos balanceadores.
- [ ] _Opcional_: Balanceo utilizando otro software como **Pound**.

## Nginx como balanceador.
En primer lugar, utilizaremos nginx como balanceador de carga. Para instalarlo en Ubuntu 16.04 ejecutamos la siguiente orden en una terminal:

```bash
  sudo apt-get install nginx
```

Una vez instalado, arrancamos el servicio desde **systemctl**.
```bash
  sudo systemctl start nginx
```

Para que nginx funcione como balanceador de carga, hay que realizar una configuración previa. El fichero de configuración por defecto 
de nginx se encuentra en _/etc/nginx/conf.d/default.conf_. Debemos eliminar el contenido de este fichero para insertar nuestra
configuración personalizada ya que, en la configuración inicial nginx funciona como servidor web. Utilizamos nuestro editor preferido
y modificamos el fichero:

```bash
  sudo nano /etc/nginx/conf.d/default.conf
```

