# Markdown
Markdown notes and practice.

## Docker notes
### Comandos

`docker ps` : Este comando se utiliza para listar todos los contenedores en ejecución en el host Docker.

`docker run` : Este comando se utiliza para ejecutar una nueva instancia de un contenedor a partir de una imagen Docker.

`docker exec`: Este comando se utiliza para ejecutar un comando en un contenedor en ejecución. Por ejemplo, puedes utilizar este comando para entrar en un contenedor y ejecutar comandos dentro de él.

`docker logs`: Este comando se utiliza para obtener los registros de un contenedor en ejecución.



#### Para interactuar con la información del contenedor
* `docker cp <container_id>:<file_path> <local_destination>`: Este comando se utiliza para copiar un archivo o directorio desde un contenedor de Docker a una ubicación en la máquina host. El comando requiere el ID del contenedor del que se va a copiar el archivo, la ruta del archivo dentro del contenedor y la ruta de destino en la máquina host donde se desea guardar el archivo
* `docker run -it -v /path/on/host:/path/in/container [image] [command]`: Este comando se utiliza para ejecutar un contenedor de Docker y compartir un directorio entre el contenedor.
* `docker volume`: Este comando se utiliza para trabajar con volúmenes en Docker. Con este comando, puedes crear, eliminar y listar volúmenes en Docker.


En este ejemplo, el flag -v se seguido por la ruta del directorio en la máquina host (/path/on/host) y la ruta del directorio en el contenedor (/path/in/container), separados por dos puntos (:). Esto permite compartir el directorio especificado en la máquina host con el contenedor de Docker.

#### Creación de imágenes

1. Se crea un archivo Dockerfile
2. Se utiliza el FROM para indicar la imagen base, de la siguiente forma: `FROM ubuntu:latest`.

3. Luego va el comando `RUN touch /ust/src/hola-platzi.txt` (comando a ejecutar en tiempo de build)

4. `docker run -it ubuntu:platzi` (corro el contenedor con la nueva imagen)
5. `docker login` (me logueo en docker hub)
6. `docker tag ubuntu:platzi miusuario/ubuntu:platzy`(cambio el tag para poder subirla a mi docker hub)
7. `docker push miusuario/ubuntu:platzi` (publico la imagen a mi docker hub)


### Uso de imágenes y capas en contenedores

1. Inicialmente se tiene un archivo DockerFile estructurado para que las dependencias se carguen primero que el código dentro del contenedor, esto para aprovechar la caché de docker al momento de hacer una build y aplicacioón de las capas en un ejemplo se puede ver de la siguiente forma:
```bash
    FROM node:12

    # Crea un directorio para la aplicación
    RUN mkdir -p /usr/src/app

    # Establece /usr/src/app como el directorio de trabajo
    WORKDIR /usr/src/app

    # Instala las dependencias
    COPY package*.json ./
    RUN npm install

    # Copia el código de la aplicación
    COPY . .

    # Expone el puerto 3000
    EXPOSE 3000

    # Inicia la aplicación
    CMD [ "npm", "start" ]
```

2.  Ejecución en modo monitor para archivos de .js en caso de correr alguna aplicación de node:
    ```bash
    CMD ["npx", "nodemon", "-L", "index.js"]
    ```
3. Esto se complementa con un bind mount para compartir un archivo entre el contenedor y la máquina; especialmente usando -v desde el sitio donde se encuentra el código al sitio donde quedará el código en el interior del contenedor.

```bash
# Primero, asegúrate de tener el archivo de código que quieres compartir en tu máquina en el directorio deseado.
# En este ejemplo, asumiremos que el archivo se llama "code.py" y se encuentra en el directorio actual (.).

# A continuación, usa el comando docker run para iniciar un contenedor de Docker y establecer un bind mount para compartir el archivo de código.
# Usa la opción -v seguida de la ruta al archivo de código en tu máquina y la ruta al directorio donde quieres que aparezca el archivo en el contenedor.

docker run -v $(pwd)/code.py:/app/code.py -it <nombre_de_la_imagen> bash

# En este caso, el archivo de código se compartirá entre tu máquina (en el directorio actual) y el contenedor (en el directorio /app).
# Cualquier cambio que realices al archivo de código en tu máquina se reflejará en el contenedor y viceversa.
# También puedes usar esta técnica para compartir otros tipos de archivos, como configuraciones o recursos, entre tu máquina y el contenedor.
```

4. Conexión entre contenedores:

    * Docker utiliza redes virtuales para la conexión de sus contenedores.

    * `docker network ls`  es el comando para mirar las redes que hay entre los contenedores.

    * `docker network create --attachable braitoNET`es el comando para crear una red en docker y el attachable es un extra para permitir que otros contenedores se conecten a ella. 
    * __NOTA__: Es posible ver el estado de las redes con el comando inspect algo tipo: `docker network inspect <nombre_red>`

    *  Con respecto a las conexiones de los contenedores a la red estas se realizan a través del comando docker network connect <nombre de la red> <nombre del contenedor>.
    ## Comandos:
    *  docker network ls (listo las redes)
    *  docker network create --atachable plazinet (creo la red)
    *  docker inspect plazinet (veo toda la definición de la red creada)
    *  docker run -d --name db mongo (creo el contenedor de la BBDD)
    *  docker network connect plazinet db (conecto el contenedor “db” a la red “platzinet”)
    *  docker run -d -name app -p 3000:3000 --env MONGO_URL=mondodb://db:27017/test platzi (corro el contenedor “app” y le paso una variable)
    *  docker network connect plazinet app (conecto el contenedor “app” a la red “plazinet”)







