# Fundamentos de Docker

+ **Problemas frecuentes al desarrollar software**
  + **Construir**: Escribir el código, depurarlo, compilarlo, etc.
    + Dependencias de desarrollo.
    + Distintas versiones de entorno de desarrollo.
  + **Distribuir**: Despliegue en servidores o entornos de ejecución.
    + Tipos de servidores.
    + Entornos nativos o virtualizados.
    + Serverless.
  + **Ejecutar:** Comportamiento de la ejecución una vez ejecutado en un entorno de ejecución.
    + Versiones de runtime.
    + Incompatibilidad del sistema operativo.
    + Recursos de hardware.



## ¿Por qué Docker?

+ Docker promete ser la solución frente a los problemas mencionados arriba ya que nos permite construir, distribuir y ejecutar una aplicación en cualquier entorno de ejecución.

+ Docker no es un servidor, sino un servicio de sistema operativo  (daemon) que corre en nuestro pc o en otro, el cliente es otro programa  que también corre en nuestro pc que le da ordenes. Es decir, el cliente  (cmd) le da las ordenes  o se comunica por back al daemon quien crea los  contenedores.

  Si corres un contenedor por primera vez, Docker lo descarga o busca  su imagen, pero si lo corres una segunda vez, ya no muestra la información y  corre lo que tiene que correr.

![goals](https://github.com/mcortegana/curso-docker/blob/master/src/goals.png)



## Qué es un contenedor?

Un contenedor una agrupación de procesos o entidad lógica () que se ejecuta de forma nativa como cualquier otra aplicación en el S.O.

### Caracteristicas

+ No consumen más recursos de lo que se preajustaron previamente en el contenedor.
+ Se ejecutan de forma nativa.
+ Su sistema de archivos toman como root "/" la ruta parcial de la ubicación del contenedor y no pueden navegar más arriba en el  árbol de archivos.
+ El único recurso  que es transparente entre el host y el contenedor es el kernel del S.O.
+ Docker solo corre  en Linux.



## Volúmenes en Docker

Siendo que el contenedor de docker esta aislado del resto del mundo,  tiene un espacio en disco reservado y además si por alguna razón hay que  reiniciar el contenedor, toda la información se habrá perdido. Lo mismo  aplica para las bases de datos, y cuando queremos tener el código fuera  del contenedor por motivos de desarrollo.

La solución a esto es decirle al contenedor donde queremos almacenar  nuestros archivos en el sistema operativo anfitrión. Para ello podemos  arrancar el contenedor con la bandera *-v <ruta_anfitrión>:<ruta_contenedor>.*

## Qué son las imágenes?

Las imágenes son plantillas de contenedores. Una imagen es inmutable; esto quiere decir que una vez es creada, esta ya no podrá cambiar.

Una imagen se crea a partir de un *Dockerfile* mediante un ***build*** y a raíz de estas imágenes es que se crean los contenedores con el comando ***run***.

![1554738862164](https://github.com/mcortegana/curso-docker/blob/master/src/1554738862164.png)

Un Dockerfile tiene la siguiente forma:

```bash
FROM ubuntu

RUN touch /usr/src/hola-miguel
```

siempre empieza con un **FROM** que es la imagen raíz por así decirlo y los  comandos **RUN** y otros que especifiquemos estarán una capa por encima de este, gráficamente hablando.

![1554740134331](https://github.com/mcortegana/curso-docker/blob/master/src/1554740134331.png)

## Dockerizando aplicaciones

Para ***dockerizar*** una aplicación (*crear una imagen de ella*), se debe hacer uso de un archivo Dockerfile; este archivo *Dockerfile* puede tener la siguiente estructura:

```dockerfile
# Dockerfile de la imagen raiz
FROM node:8
# Ordena que se deben copiar todos los archivos del contexto actual del build
# [<ubicación de los archivos a copiar>,<destino>]
COPY [".","/usr/src"]
# Especifica el  directorio donde se ubican los archivos de nuestra aplicación y navega hacia ella.
WORKDIR /usr/src
# Ejecuta el comando npm install
RUN npm install
# Indica el puerto de salida o puerto por el  cual  se expondrá el contenedor.
EXPOSE 3000
# Especifica un comando por defecto que se ejecutará al crear un contenedor
# en este caso se ejecuta el comando "node index.js" que inicializa un servidor http de
# nodejs con express.
CMD ["node","index.js"]
```

Para construir la imagen usamos el comando:

+ ***docker build -t "nombre:version"*** .

[best-practices]:https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

En la [documentación][best-practices] se puede encontrar una guía de referencia con  las mejores practicas al  momento de escribir Dockerfiles.

### Optimizar la cache de construcción de Docker

Docker utiliza una memoria cache con la idea de agilizar la construcción de imágenes. Como se menciono antes las imágenes son construidas por capas; en un  archivo Dockerfile cada instrucción se traduce en una capa de la imagen.

Docker puede reutilizar, siempre que se pueda, las capas de una construcción anterior. Evitando de esta forma volver a ejecutar instrucciones que ya han sido utilizadas previamente; para esto hay que tener en cuenta algunas cosas al momento de construir un *Dockerfile*.

+ Colocar las instrucciones del *Dockerfile* que **tienden a cambiar** en la parte final  del fichero. Dado que si están entre las primeras líneas, *Docker* reconocerá un cambio e invalidará todo el  cache de una construcción anterior que podíamos haber reutilizado.

+ Agrupar instrucciones en una misma capa o instrucción del fichero *Dockerfile*. Se muestra mejor en el  siguiente ejemplo:

  ​	

  ```dockerfile
  FROM debian:9
  RUN apt-get update
  RUN apt-get install -y nginx
  
  # En lugar su lugar, es mejor combinar ambas instrucciones RUN en una sola.
  
  FROM debian:9
  RUN apt-get update
  	apt-get install -y nginx
  ```

[construccion]: https://medium.com/@serrodcal/buenas-pr%C3%A1cticas-construyendo-im%C3%A1genes-docker-8a4f14f7ad1d

Estas y más recomendaciones al momento de construir imágenes docker y ficheros *Dockerfile* podemos encontrar en el  siguiente [blog][construccion].

## Redes en Docker

*Docker* proporciona los fundamentos y herramientas necesarias para la creación de redes y comunicación entre contenedores y contenedor-host. Permite crear *subredes* y unir contenedores a esta para que trabajen conjuntamente.

Por defecto *Docker* tiene 3 redes disponibles:

+ **bridge**: red por defecto, actualmente en desuso.
+ **host:** Simula totalmente la red del host o computador que esta corriendo *Docker*, se recomienda no usar.
+ **none:** Aísla o deshabilita la red.

Para crear una nueva red sobre la que podamos hacer que 2 o más contenedores colaboren se usa el  comando:

+ **docker network create --attachable \<nombre-de-red\>**

El  *flag* **--attachable** indica que otros contenedores se pueden unir a esta red.

Una vez creada esta red podemos hacer que 2 o más contenedores se unan a ella mediante el comando:

**docker network connect \<nombre-red\> \<nombre-contenedor\>**

Ejemplo:

**docker network create --attachable dockernet** => Creamos la red *dockernet*.

**docker run -d --name mongodb mongo** => Creamos un contenedor de mongo con el nombre *mongodb.*

**docker network connect dockernet mongodb** => Une el contenedor mongodb a la red *dockernet*.

**docker run -d --name app -p 3000:3000 --env MONGO_URL=mongodb://mongodb:27017/test dockerapp** => Especifica mediante una variable de entorno *MONGO_URL* la cadena de conexión hacia la base de datos mongo unidad a la red *dockernet*, esta variable de entorno se usará para realizar la conexión hacia la base de datos mongodb.

Solo quedaría unir el contenedor app a la red dockernet y listo.

**docker network connect dockernet app**

## Docker Compose

***Docker Compose*** es una herramienta que permite simplificar el uso de Docker, generando scripts que facilitan el diseño y la construcción de servicios, permitiéndonos  describir de forma declarativa la arquitectura de nuestra aplicación utilizando *composefile* (docker-compose.yml).

Un fichero *docker-compose.yml* tiene la siguiente estructura.

```yaml
version: "3"

services:
  app:
    image: demoapp
    environment:
      MONGO_URL: "mongodb://db:27017/test"
    depends_on:
      - db
    ports:
      - "3000:3000"

  db:
    image: mongo
```

*Docker Compose* creará los contenedores y redes necesarios según estén declarados en el archivo *.yml*

Para crear los servicios con *Docker Compose*, usar el comando:

***docker-compose up -d***, el *flag* **-d** indica que no mostrará la salida de consola del contenedor.

Para detener los servicios en ejecución:

***docker-compose down,*** esto detendra los servicios ejecutados.

Tambien podemos acceder a un contenedor con:

***docker-compose exec \<nombre-servicio\> \<bash ó alguna ejecución\>*** 

##### Comandos Útiles

+ ***docker ps -a*** = lista a detalle todos los contenedores.
+ ***docker ps -aq*** = lista todos los id´s de los contenedores.
+ ***docker run "nombre-imagen"*** = Corre un contenedor de una imagen, si no  existe lo descarga.
+ ***docker run --name "nombre que queremos darle al  contenedor" "nombre-imagen"*** = Crea un contenedor de una  imagen con un nombre específico dado como parámetro.
+ ***docker inspect "id-contenedor"*** ó ***docker inspect "nombre-contenedor"*** = Muestra los detalles internos del contenedor.
+ ***docker rm "nombre-contenedor"*** = Elimina el contenedor con un nombre especifico.
+ ***docker rm -f "nombre-contenedor"*** = Fuerza la eliminación de un contenedor con un nombre especifico.
+ ***docker rm "id-contenedor"*** = Elimina el contenedor con el id indicado.
+ ***docker rm $(docker ps -aq)*** = Elimina todos  los contenedores.
+ ***docker run -it "nombre-imagen"*** = Corre un contenedor de una imagen en modo interactivo (Usando el  terminal).
+ ***docker exec "nombre-contenedor"*** = Corre un contenedor ya existente, solo no esta en estado "EXITED".
+ ***docker kill "nombre-contenedor"*** = Mata el grupo de procesos de un contenedor
+ ***docker volume create "nombre-del-volumen"*** =  Crear un volumen.
+ ***docker run -d --name "nombre-del-contenedor" --mount src="nombre-volumen",dst="carpeta-datos-contenedor" "nombre-de-imagen"*** = Crea un contenedor con un nombre específico y además asigna un volumen de almacenamiento a una ruta usada por la aplicación.
+ ***docker volume prune*** = elimina los volúmenes que no estén en uso.
+ ***docker volume ls*** = lista los volúmenes existentes.
+ ***docker volume rm "nombre-de-volumen"*** = Elimina un volumen específico.
+ ***docker run -d  -p 8080:80 nginx*** = El flag **-p** vincula un puerto del contenedor a un puerto local en la forma *<puerto_local>:<puerto_contenedor>*.
+ 