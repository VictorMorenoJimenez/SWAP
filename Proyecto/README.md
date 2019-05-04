# Desplegar clúster de Kubernetes en servidores de Hetzner Cloud

## Autores
Moreno Jiménez, Víctor

Pozo Tena, Pablo

## Introducción
Docker y en general el software de contenedores está de moda en el mundo del software. Hoy en día las grandes compañías están adoptando un estilo de trabajo DevOps y Docker permite que estos equipos de desarrollo consigan una alta eficiencia en la entrega y despliegue de su software. Gracias a Docker podemos separar los distintos componentes o servicios de una aplicación en entornos aislados donde no habrá problemas de compatibilidad con otro software. Sin embargo éstos contenedores pueden comunicarse con facilidad. En este punto es donde entra en juego Kubernetes, que es un software de código abierto para administrar aplicaciones en contenedores. Kubernetes esta diseñado para agrupar los contenedores en unidades lógicas para su fácil administración.

El propósito de este documento es ilustrar como desplegar una aplicación Django dockerizada sobre un clúster con Kubernetes. Primeramente desplegaremos un clúster sobre Hetzner Cloud orquestado con Kubernetes dónde desplegaremos la aplicación Django. 

Para comprender el propósito de este documento así como su puesta en práctica hay que conocer una série de conceptos previos que son indispensables para entender el proyecto.

## Conceptos previos

### Kubernetes
Para definir correctamente Kubernetes debemos tener en cuenta el conceto de orquestador. En el ámbito del software, un orquestador es un programa que maneja interconexiones e interacciones entre cargas de trabajo en nubes públicas y privadas. Un orquestador se encarga de conectar las tareas automatizadas en un flujo de trabajo para cumplir las metas con vigilancia de permisos y aplicación de políticas concretas. Esto es, en esencia, Kubernetes un gran director de orquesta que dirige y gobierna los contenedores. Dicho así parece muy sencillo y así es pero para entender Kubernetes hay que entender su arquitectura y como se las arregla para poder administrar, monitorizar y interconectar los diferentes contenedores.

### Contenedor
Como hemos explicado anteriormente, Kubernetes es un software de orquestación de contenedores. Esto nos conduce a la siguiente pregunta, ¿qué es un contenedor?

Un contenedor no es más que un sistema aislado con un conjunto de tecnologías que permiten ofrecer un servicio en su totalidad. Los contenedores aportan modularidad y previenen errores de compatibilidad ya que una vez creado el contenedor puede correr en cualquier servidor. Por ejemplo, si queremos pasar una aplicación a contenedores debemos separar los distintos servicios que ofrece la aplicación en contenedores. Un contenedor para el frontend, otro para el backend y un último para la base de datos. Dentro del contenedor se especifican las necesidades particulares de cada servicio. Por ejemplo en el contenedor de base de datos si se usa mariadb, crearemos un contenedor con una imágen base de mariadb y los parámetros de configuración necesarios. 

A continuación se muestra como ejemplo la arquitectura de Docker y sus contenedores. Fíjese que Docker no virtualiza hardware como haría una máquina virtual si no que corre sobre el mismo sistema anfitrión lo que lo hace mucho más liviano. A partir de ahí una vez instalado Docker en la máquina se pueden desplegar tantos contenedores como las prestaciones del servidor admitan.

![Arquitectura Docker](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/Proyecto/img/dockerArchitecture.png)

Esta imágen ilustra perfectamente el funcionamiento de Docker. De izquierda a derecha:
* Docker_Client


 Estos son los comandos básicos para trabajar con Docker.
 Build para construir la imagen
 Pull para bajar una imagen del registry
 Run para correr una imagen ya creada.
* Docker_Host


 En este apartado tenemos un ejemplo de Docker corriendo en un equipo con varios contenedores.
 Cada contenedor se puede construir con una imágen diferente creada o descargada de algún registro de Docker o privado.
* Registry


 El registry es un repositorio donde se almacenan las imágenes de Docker. 
 Los registry pueden ser privados o públicos. El registry más famoso de Docker es docker-hub donde la comunidad puede subir sus   imágenes personalizadas.


En este proyecto se ha elegido Docker como software de contenedores y se explicará más adelante como se ha dockerizado la aplicación Django.

Ahora ya conocemos los conceptos básicos de lo que es un contenedor y unas pinceladas de como funciona Docker. El propósito de este documento no es una guía extensa de Docker. Si se quiere profundizar en el tema puede consultar la documentación oficial.

### Arquitectura Kubernetes
Anteriormente hemos presentado Kubernetes como un software de orquestación de aplicaciones para administrar y monitorizar los contenedores. Esto está muy bien pero, ¿cómo consigue Kubernetes realizar esto? 

Antes de nada vamos a explicar unos conceptos previos necesarios para la explicación posterior:

* Clúster

Conjunto de servidores o máquinas, físicas o virtuales, que utiliza Kubernetes.

* Pod

Un Pod es a Kubernetes lo que un contenedor es a Docker. Por ejemplo, cuando nosotros despleguemos nuestra aplicación, el contenedor asociado a la base de datos, pasará a ser un pod en Kubernetes.

* Replica

Kubernetes nos permite crear réplicas de nuestros pods de manera sencilla gracias a los deployments.

* Deployment

Un deployment usualmente es un fichero .yaml que el nodo maestro utilizará para desplegar los pods. En los ficheros deployment le indicamos al máster la cantidad de recursos que el pod en cuestión necesitará o cuantas réplicas queremos desplegar. Por ejemplo, desplegando un servidor nginx le podemos decir al máster a través de el fichero deployment que el pod ha de tener 4GB de ram y 40GB de disco. El nodo máster se encargará de que no le falten recursos al pod.

* Node

Un nodo en Kubernetes es una máquina "esclava o trabajadora". Usualmente un nodo corresponde con una máquina física y cada nodo contiene los servicios necesarios para correr sus "pods" y es gobernado por el nodo máster.
* Service

Un servicio en Kubernetes se puede definir como la gestión de acceso a los pods y abstracción de pods. Como ejemplo podemos considerar un servidor backend con 3 replicas. El pod con el frontend no debería preocuparse de si el backend cambia porque el maestro lo ha decidido. Aquí es donde entran los servicios que permite a los pods despreocuparse.



### ¿Cómo funciona Kubernetes?

Como ya sabemos, Kubernetes se encarga de gestionar un conjunto de contenedores facilitando el acceso y monitorización de cara a los administradores del sistema. Kubernetes engloba un conjunto de máquinas y facilita una API para que el administrador pueda gestionar los nodos.


![Arquitectura Kubernetes](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/Proyecto/img/kubernetesarchitecture.png)

Esta imágen define perfectamente el funcionamiento de Kubernetes. Empecemos explicando un poco los componentes...
* Master

El nodo Master realmente es un conjunto de 3 procesos que se encargan de cumplir los requisitos de cada uno de los pods. 
Estos tres procesos son: Kubernetes kube-controller-manager, kube-apiserver y kube-scheduler.

* Api Server

Se encarga de validar y configurar la información de los objetos tales como pods, servicios, replica controllers... Forma parte del nodo Master.


* Scheduler

Scheduler se encarga de hacer cumplir los requisitos de cada pod previamente introducidos por la API. También se encarga de distrbuir las réplicas de los pods por el clúster. Si el scheduler falla al intentar cumplir los requisitos de un pod, intentará reorganizarlo hasta que la máquina esté disponible.

* Controller Manager

Es un daemon o demonio que agrupa el núcleo principal de control de Kubernetes. Es un bucle constante que se encarga de revisar el estado del cluster a través de la API server y hace los cambios convenientes para pasar del estado actual al estado deseado. Algunos ejemplos de controladoers son: replication controller, endpoint controller, namespace controler...


* Kubelet

El kubelet es la unidad primaria de Kubernetes. Cada nodo posee un agente llamado kubelet. Kubelete abstrae la información de requisitos de un pod a través de la API server y se encarga de que lo descrito en estos ficheros .yaml de configuración esté en sintonía con el estado del pod. 

* Proxy

Para manejar el subnetting de los hosts individuales y hacer posible la conectividad entre otros servicios, un pequeño servicio de proxy se crea en cada nodo. Este proceso se encarga de hacer balanceo de carga y de hacer accesible el nodo.

Ahora que ya conocemos los conceptos básicos vamos a ver como interactúan entre ellos: 
De forma general, el Master es el responasble de administrar todo el clúster. Cada nodo (ya sea una máquina física, virtual o varias máquinas físicas o virtuales), posee un Kubelet que es el agente que va a realizar las comunicaciones con el Master.  Cuando se despliega una aplicación en un clúster de Kubernetes, se informa al Master a través de los ficheros .yaml llamados deployment. El Master se encarga de distribuir estos contenedores a través del clúster y de cumplir con los requisitos especificados en estos ficheros de configuración .yaml(requisitos de memoria RAM, CPU, memoria...). A su vez los nodos se comunican con el Master usando la Kubernetes API que el Master ofrece.  

Desde un punto de vista más cercano, diferenciando los 3 procesos de el nodo Master lo que ocurre al crear un deployment es lo siguiente:

* kubectl (la forma de comunicarse con el Master por línea de comandos) escribe en la API server.
* API server valida la información y la almacena en Cluster store(etcd)
* etcd le devuelve la confirmación a API server.
* API server invoca al scheduler.
* Scheduler se encarga de decidir en que servidores va a desplegar los pods y le devuelve el control al API server.
* API server lo almacena en etcd.
* etcd manda confirmación a la API server.
* API server invoca los kubelets necesarios segun los nodos desplegados.
* Kubelet se comunica con el Docker daemon utilizando la API sobre un socket de Docker para crear el contenedor.
* Kubelet actualiza el estado del pod recién creado e informa a la API server.
* API server almacena el nuevo estado en etcd.

Ahora que ya conocemos como funciona Kubernetes, sus conceptos básicos, y conocemos la arquitectura de Docker en su forma más básica estamos preparados para comenzar con el despliegue de la aplicación en un clúster de kubernetes. 

## Paso 1. Dockerizar aplicación Django
Como condición inicial para desplegar un clúster de Kubernetes, debemos tener dockerizada la aplicación Django. Como ejemplo hemos elegido una aplicación desarrollada por Intelligenia S.I empresa en la que actualmente Víctor Moreno Jiménez está realizando las prácticas de empresa y Pablo Pozo Tena trabaja en el puesto de sysadmin.

Para dockerizar copicloud hemos separado la aplicación en tres bloques:
* Frontend: aplicación Angular.
* Backend: aplicación Django.
* Base de datos: mariadb.

Para empezar debemos realizar los correspondientes Dockerfiles para cada una de éstas partes. No profundizaremos mucho en como se han realizado los Dockerfiles simplemente añadir que en éstos se indica como instalar las dependencias correspondientes de cada aplicación y las directivas para cuando se ejecute el contenedor.

Aqui se muestra el primer Dockerfile para la aplicación Angular. 

``` bash
# base image
FROM node:9.6.1 as builder

# set working directory
RUN mkdir /usr/src/app
WORKDIR /usr/src/app

# add `/usr/src/app/node_modules/.bin` to $PATH
ENV PATH /usr/src/app/node_modules/.bin:$PATH

# install and cache app dependencies
COPY package.json /usr/src/app/package.json
RUN npm install grunt-cli karma@0.10 bower
RUN npm install


# add app
COPY . /usr/src/app

# generate build
RUN grunt compile -f

##################
### production ###
##################

# base image
FROM nginx:1.13.9-alpine

# copy artifact build from the 'build environment'
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder /usr/src/app/bin /usr/share/nginx/html


# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]

```

El Dockerfile de el backend contiene las instrucciones para desplegar un entorno python con las instrucciones para instalar todas las dependencias que necesita el servidor de copicloud para gestionar los documentos. 

```bash
FROM python:2.7-stretch

ENV PYTHONUNBUFFERED 1

RUN echo "deb http://www.deb-multimedia.org stretch main non-free" >> /etc/apt/sources.list
RUN apt-get update && apt-get install python-dev -y
RUN apt-get install libmariadbclient-dev -y
RUN apt-get install deb-multimedia-keyring -y --allow-unauthenticated

RUN dpkg --add-architecture i386; apt-get update ;\
    apt-get install -y  acroread


RUN apt-get install -y libjpeg-dev libfreetype6-dev zlib1g-dev libtiff-dev libwebp-dev liblcms2-dev graphviz-dev graphviz pkg-config libxml2-dev libxslt1-dev
RUN apt-get install -y ghostscript texlive poppler-utils imagemagick libreoffice-writer libreoffice-impress

RUN rm -rf /var/lib/apt/lists/*


# Requirements are installed here to ensure they will be cached.
COPY ./src/requirements.txt /requirements.txt
RUN pip install --no-cache-dir -r /requirements.txt \
    && rm -rf /requirements.txt

COPY ./start.sh /start.sh
RUN sed -i 's/\r//' /start.sh
RUN chmod +x /start.sh

COPY ./entrypoint.sh /entrypoint.sh
RUN sed -i 's/\r//' /entrypoint.sh
RUN chmod +x /entrypoint.sh

COPY ./src /app

WORKDIR /app

ENTRYPOINT ["/entrypoint.sh", "/start.sh"]

```

Por último la base de datos no necesitará de un Dockerfile ya que al no tener ningún tipo de dependencia especial se utilizará la imágen mariadb:latest descargada directamente de los repositorios de docker-hub. 


Una vez ya tenemos todos nuestros Dockerfiles creados debemos construir las imágenes con el comando docker build y una vez construida la imágen procederemos a subirla a nuestro registro privado de imágenes configurado en gitlab. Ésto se muestra en el siguiente video.

[![buildimaage](https://asciinema.org/a/xqNHk4wCl5hkzS5nIXM8rqeHz.svg)](https://asciinema.org/a/xqNHk4wCl5hkzS5nIXM8rqeHz)

Lo que ocurre en el video anterior es:
* Entramos en nuestro registry privado en gitlab.
* Construimos la imagen a partir del Dockerfile.
* Subimos la imagen a el registry para acceder a el desde Kubernetes.

Ahora si, todos los ingredientes están listos para desplegar el clúster de Kubernetes en los servidores Hetzner Cloud. Lo que sigue a continuación requiere de tener contratado el servicio de Hetzner Cloud y poder crear máquinas en el. Éste servicio no es gratuito y se factura por tiempo.


## Paso 2. Desplegar clúster de kubernetes en el servidor de Hetzner.
### Servidores Hetzner Cloud.
Como hemos comentadio anteriormente, para el propósito de este documento hemos adquirido dos máquinas CX21 en los servidores de Hetzner Cloud. Esto es imprescindible ya que para desplegar un clúster de Kubernetes necesitamos varias máquinas.

### Script para lanzar clúster de Kubernetes.
Para lanzar el clúster de Kubernetes nos hemos ayudado de un proyecto de la comunidad basado en kubespray subido a github __hkube__ :

[hkube]: https://github.com/bbelky/hkube

Éste proyecto permite desplegar el clúster de Kubernetes en servidores Hetzner con muy poca configuración. Primero clonamos el repositorio:

```bash
 git clone https://github.com/bbelky/hkube
 sudo nano config.json
```

![hkube conf](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/Proyecto/img/hkubeconf.png)

* hcloud_token: token de la api de hetzner.
* hcloud_user_sshkey_name: la clave ssh introducida en hetzner.
* hcloud_server_type: tipo de servidor contratado.
* hcloud_name: nombre del proyecto en hetzner cloud.
* hcloud_count: número de servidores a desplegar.

Con esta breve configuración estamos casi listos para realizar el despliegue de Kubernetes. Antes de nada debemos cargar la configuración con el siguiente comando:

```bash
 ./hkube config
```

### Despliegue clúster Kubernetes
Una vez cargada la configuración, está todo preparado para lanzar nuestro clúster de Kubernetes sobre hetzner cloud.
El siguiente comando cargará la configuración introducida anteriormente para poder desplegar los servidores correctamente. Ahora simplemente lanzamoe el script de despliegue:

```bash
 ./hkube deploy
```

Éste script creará automáticamente en hetzner dos máquinas y desplegará sobre las dos el clúster de Kubernetes. El proceso de despliegue suele tardar unos minutos, a continuación se muestra una imágen con la finalización del proceso y las máquinas hetzner corriendo.

![hetzner paso1](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/Proyecto/img/hetznerPaso1.png)

Transcurridos unos minutos el clúster de Kubernetes se ha desplegado con éxito!

![hetzner paso2](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/Proyecto/img/hetznerPaso2.png)


### Acceso a las máquinas Kubernetes
Como tenemos nuestra clave pública añadida en hetzner cloud, podemos acceder a las máquinas via ssh. Hacemos ssh a la máquina 1 que es el Master y comprobamos que todo se ha desplegado correctamente.

![Kubernetes 1](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/Proyecto/img/primeraKubernetes.png)

Comprobamos que tenemos los dos nodos activos, y que el servicio de ClusterIP. También con el comando kubectl get pod, vemos que no hay ningun pod disponible ya que no hemos creado nada aún. Simplemente tenemos los dos nodos activos, uno en cada máquina Hetzner y los servicios básicos de Kubernetes. 

## Paso 3. Crear deployments para los contenedores.
Llegados a este punto, debemos crear los ficheros .yaml conocidos como deployments. Como ya sabemos estos ficheros guardan la configuración de los pods y el Master se encarga de que esto se cumpla. Podríamos crear todos los deployments en un mismo fichero separados por tres guiones pero por modularidad y separación se ha preferido hacer en ficheros separados. 

Para empezar mostramos el fichero que se encarga de desplegar el contenedor del frontend. Le indicamos al Master en que puerto corre el pod, el número de replicas y también de donde descargar la imágen del contenedor. Antes de poder descargar la imagen del registro privado debemos iniciar sesión con el registry privado, lo veremos más adelante.

```bash
 cat frontend.yaml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: copicloud-frontend
spec:
  selector:
    matchLabels:
      app: copicloud-frontend
  replicas: 1
  template:
    metadata:
      labels:
        app: copicloud-frontend
    spec:
      containers:
      - name: copicloud-frontend
        image: registry.git.intelligenia.com/autoprinter/copicloud/frontend_last
        ports:
        - containerPort: 80
      imagePullSecrets:
      - name: regcred
```

El siguiente deployment de desplegar el pod para el backend.

```bash
 cat backend.yaml
```
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: copicloud-backend
spec:
  selector:
    matchLabels:
      app: copicloud-backend
  replicas: 2
  template:
    metadata:
      labels:
        app: copicloud-backend
    spec:
      containers:
      - name: backend
        image: registry.git.intelligenia.com/autoprinter/copicloud/backend
        ports:
        - containerPort: 8000
      imagePullSecrets:
      - name: regcred
```

Por último el deployment para la base de datos mariadb. Éste lo creamos a partir de un contenedor genérico de mariadb localizado en los repositorios oficiales de dockerhub.

```bash
 cat mariadb.yaml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - image: mariadb:latest
        name: mariadb
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: copicloud
        - name: MYSQL_USER
          value: copicloud
        - name: MYSQL_PASSWORD
          value: copicloud
        - name: MYSQL_DATABASE
          value: copicloud
        ports:
        - containerPort: 3307
          name: mariadb
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mariadb
      volumes:
      - name: mariadb-persistent-storage
        persistentVolumeClaim:
          claimName: mariadb-pv-claim
```

Es importante destacar que en este caso se utilizan volumes con data persistente en el tiempo, es decir que no se borran en caso de que se borre el pod. Para ello tenemos que crear un volumen persistente:

```bash
 cat mariadb-pv.yaml
```

```bash
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mariadb-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

El siguiente paso será iniciar sesion en el registry si tenemos la imagen en un registry privado.

```bash
 docker login registry.privado.dominio
```

Al iniciar sesión Kubernetes nos va a crear un fichero de configuración en .docker/config.json con la clave del registor privado.

```bash
  root@node1:~# cat .docker/config.json 
 {
  "auths": {
   "registry.git.intelligenia.com": {
    "auth": "*********************************"
   }
  },
  "HttpHeaders": {
   "User-Agent": "Docker-Client/18.09.5 (linux)"
  }
 }
```

Para permitir a los deployment utilizar información sensible como contraseñas de acceso, Kubernetes utiliza los secrets.
Vamos a crear un secret asociado al fichero de configuración anterior para que pueda ser usado en los deployments que necesiten acceder a la imagen del registry.

![Secreto](https://raw.githubusercontent.com/VictorMorenoJimenez/SWAP/master/Proyecto/img/crearSecreto.png)

Hemos creado un secreto llamado regcred que se utilizará en los deployment.

### Resultado final
Mostrar la funcionalidad que la aplicación django funciona correctamente dockerizada en el clúster de Kubernetes



