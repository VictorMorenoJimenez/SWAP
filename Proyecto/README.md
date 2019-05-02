# Desplegar contenedores Docker en Hetzner Cloud orquestado con Kubernetes
## Introducción
Docker y en general el software de contenedores está de moda en el mundo del software. Hoy en día las grandes compañías están adoptando un estilo de trabajo DevOps y Docker permite que estos equipos de desarrollo consigan una alta eficiencia en la entrega y despliegue de su software. Gracias a Docker podemos separar los distintos componentes o servicios de una aplicación en entornos aislados donde no habrá problemas de compatibilidad con otro software. Sin embargo éstos contenedores pueden comunicarse con facilidad. En este punto es donde entra en juego Kubernetes, que es un software de código abierto para administrar aplicaciones en contenedores. Kubernetes esta diseñado para agrupar los contenedores en unidades lógicas para su fácil administración.

El propósito de este documento es ilustrar como desplegar una aplicación Django dockerizada sobre un clúster con Kubernetes. Primeramente desplegaremos un clúster sobre Hetzner Cloud orquestado con Kubernetes dónde desplegaremos la aplicación Django. 

Para comprender el propósito de este documento así como su puesta en práctica hay que conocer una série de conceptos previos que son indispensables para entender el proyecto.

## Conceptos previos
### Contenedor
Explicar concepto de contenedor y principales software de contenerización.

### Kubernetes
Para definir correctamente Kubernetes debemos tener en cuenta el conceto de orquestador. En el ámbito del software, un orquestador es un programa que maneja interconexiones e interacciones entre cargas de trabajo en nubes públicas y privadas. Un orquestador se encarga de conectar las tareas automatizadas en un flujo de trabajo para cumplir las metas con vigilancia de permisos y aplicación de políticas concretas. Esto es, en esencia, Kubernetes un gran director de orquesta que dirige y gobierna los contenedores. Dicho así parece muy sencillo y así es pero para entender Kubernetes hay que entender su arquitectura y como se las arregla para poder administrar, monitorizar y interconectar los diferentes contenedores.

#### Arquitectura Kubernetes
Explicar los conceptos claves de kubernetes como por ejemplo:
Cluster
Master
Workers
CoreDNS
y Mucho mas

### ¿Cómo funciona Kubernetes?
Concepto de cluster de desplegar cluster de kubernetes en una maquina.
Explicar el flujo de control de kubernetes. Como el master va controlando los workers.
Como los deployments los ejecuta el master cada vez q falla algo.
Como el master es capaz de reorganizar los pods segun sus necesidades....
