# Práctica 3. Balanceo de carga.

**Objetivos** de la práctica 3:

- [ ] Configurar una máquina e instalarle el **nginx** como balanceador de carga.
- [ ] Configurar una máquina e instalarle el **haproxy** como balanceador de carga.
- [ ] Comprobar funcionamiento de algoritmos de balanceo como **round-robin** _Peso M1 = 2*Peso M2_.
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

Para una configuración inicial, sencilla editamos el fichero como aparece a continuación:

![iniConf](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/confInicial.png)

El primer campo **upstream** indica los servidores apaches que dispone el balanceador. A continuación con la directiva
server se incluyen una serie de parámetros como el puerto de escucha, el nombre del servidor, donde almacenar los logs etc.
Antes de reiniciar el servicio debemos desactivar la configuración por defecto que viene con nginx. Para ello borraremos el fichero de configuración siguiente:

```bash
  sudo rm /etc/nginx/sites-enabled/default
```
Si no te gusta la solución anterior, también puedes comentar la línea del fichero _nginx.conf_ que habilita nginx como servidor web. De ésta manera únicamente se cargará la configuración _/etc/nginx/conf.d/default.conf_. Ambas opciones funcionan.

Para comprobar que el servicio está funcionando correctamente lanzamos el comando:

```bash
  sudo systemctl restart nginx
```

Si no aparece ningún error parece que el fichero de configuración está correctamente. Vamos a probar lanzando desdde el **host** dos órdenes _curl_ para comprobar que el balanceo funciona correctamente. Importante que el _index.html_ de _M1_ y _M2_ sea diferente para diferenciar cuando responde _M1_ y cuando _M2_. En la siguiente foto se muestra una petición de __curl__ a la máquina _balanceadora_, comprobamos como efectivamente hace una petición a cada máquina. Funciona como balanceadora de carga!


![balanceTest](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/curlBalancer.png)

Llegados a este punto hemos cumplido el primer objetivo:

- [X] Configurar una máquina e instalarle el **nginx** como balanceador de carga.

A continuación vamos a probar a darle peso a las máquinas de manera que _Peso M1 = 2*Peso M2_. Para ello debemos modificar el fichero de configuración y hacer unas pequeñas modificaciones.

```bash
  sudo nano /etc/nginx/conf.d/default.conf
```

```bash
  upstream apaches {
  server 192.168.56.50 weight=2;
  server 192.168.56.51 weight=1;
}
```
De ésta forma con la orden _weight_ le estamos dando más importancia a la máquina 1. Con ésta configuración cada 3 peticiones 2 irán para la máquina 1 y 1 para la máquina 2. Cargamos el servicio nginx para que surjan efecto los cambios...

```bash
  sudo systemctl reload nginx
```
Y probamos la nueva configuración realizando 3 peticiones _curl_.

![balanceTest](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/curlPesos.png)

Una vez comprobado que funciona el algoritmo _round robbin_ con pesos, vamos a someter a nuestro balanceador nginx a una alta carga utilizando el software __apache benchmark__. Para ello lo instalaremos en el host y lanzaremos una alta carga al balanceador. __Apache benchmark__ es una herramienta muy sencilla que nos ayudará a poner a prueba nuestro servidor web, en este caso nuestro balanceador de carga. Su instalación en _ubuntu_ es muy sencilla:

```bash
  sudo apt-get install apache2-utils
```

Una vez instalado vamos a hacer 3 pruebas de carga con nuestro balanceador nginx: Primero lanzaremos una carga pequeña de unas 100 peticiones, segundo una carga media de unas 1000 peticiones y por último una carga alta de unas 10000 peticiones. El parámetro _-c_ nos indica la concurrencia que la dejaremos en 10 para todas las pruebas. Desde el host ejecutamos las 3 pruebas y analizamos los resultados:

```bash
  ab -n 100 -c 10 http://192.168.56.52/index.html
```
Resultados:

![resultados100](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/stats100.png)

```bash
  ab -n 1000 -c 10 http://192.168.56.52/index.html
```
Resultados:

![resultados1000](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/stats1000.png)

```bash
  ab -n 10000 -c 10 http://192.168.56.52/index.html
```
Resultados:

![resultados10000](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/stats10000.png)


Éstos resultados los utilizaremos para compararlos con los tiempos de **haproxy** el próximo balanceador a comparar. Ya hemos terminado con __nginx__ con lo cual deberemos desactivar el servicio ya que corren en el mismo puerto. Para ello:


```bash
  sudo systemctl stop nginx
```

Y procedemos a la instalación de **haproxy**. En ubuntu una vez más es muy sencillo:

```bash
  sudo apt-get install haproxy
```

Una vez instalado debemos modificar el archivo de configuración _/etc/haproxy/haproxy.cfg_ igual que hicimos con nginx.

```bash
  sudo nano /etc/haproxy/haproxy.cfg
```
Borramos la configuración que viene por defecto. El fichero final debe quedar como el de la imágen.

![haproxyConf](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/haproxyConf.png)

Con ésta configuración le indicamos a haproxy quienes son las dos máquinas servidoras web finales, m1 y m2, que escuchan por el puerto 80. También se definen otros parámetros como el timeout de la conexión, del cliente y del servidor. Para comprobar el funcionamiento de haproxy primero lo lanzamos indicandole el fichero de configuración:

```bash
  sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg
```

A continuación desde el host procedemos de igual forma que con nginx, vamos a lanzar peticiones _curl_ para observar si el balanceo lo ejecuta correctamente:

![balanceoHaproxy](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/P3/img/balanceoHaproxy.png)








