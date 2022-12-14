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






