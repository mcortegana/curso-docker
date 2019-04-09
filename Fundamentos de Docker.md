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

+ ***docker build -t "nombre:version"***

[Mejores prácticas al escribir  ficheros Dockerfile]:https://docs.docker.com/develop/develop-images/dockerfile_best-practices/



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