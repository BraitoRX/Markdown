# HTTPS 
Instrucciones para garantizar HTTPS en GCP con un proyecto DJANGO.

## Ingredientes

- NGINX con UWSGI
    ![alt text](https://tonyteaches.tech/wp-content/uploads/2020/10/nginx-uwsgi-django-stack-1536x128.png) 
    >**NGINX**: Nginx es un servidor web que también puede ser usado como proxy inverso, balanceador de carga. 
    Su principal función es lograr una conexión con el socket de unix para conectar con los puertos adecuados para qué la petición que se envía redireccione al puerto adecuado, él maneja también los certificados del dominio para lograr HTTPS.

    >**uWSGI**: Servidor de aplicaciones que se encarga de recibir las peticiones que recibe NGINX y ejecutarlas en el entorno virtual de python sobre el proyecto Django asociado, este módulo es capaz de resolver las consultas y escalar de ser necesario para resolverlas a todas, también se puede usar Gunicorn en su defecto y el código no cambia, es decir, se comporta de forma modular con cualquier otro servidor de aplicaciones de python.
- Entorno virtual de Python 
    > Se necesita un entorno virtual para que el manejo de dependencias de un proyecto DJANGO sea independiente y no se instalen directamente en la máquina sino en un entorno donde se tiene completo control sobre los módulos importados, estos a su vez son usados por uWSGI para decidir en qué ambiente tiene que ejecutar la petición HTTP/HTTPS.
- Ejecución en un entorno de producción
    > El gran problema que tienen los servidores de aplicaciones que tienen implementados Django y otras tecnologías de backend, es que corren en modo de prueba y por lo tanto no es posible monitorear ciertos comportamientos que se desean en una app desplegada propiamente, el entorno de producción hace posible que la aplicación corra en segundo plano con sus capacidades al 100 %, esto incluye manejo de peticiones HTTPS, para aplicaciones web Python el modelo usado es NGINX conectado a uWSGI para el manejo de entornos de producción.
- Dominio: 
    > Deberá crear un un dominio y tener presente los certificados de este en la misma máquina virtual, estos los puede obtener a través de ZeroSSL para el certificado y freenom para el dominio.
    ![imagen_prueba](https://i.gyazo.com/a45a219e062d587a0f7b9557c5ec4a26.png)
    Luego le dará en finalizar compra y si el dominio no se buguea tendrá lo llevará a una página donde podrá terminar su compra por tan solo 0$.

- Configuración CLOUD DNS GCP

    > Para que los dominios mapeen la dirección ip a la que tienen que redirigir estén correctamente definidos es necesario que se cree un DNS que reconozca la dirección y mapee esta a la dirección IP de la máquina o balanceador asociado con el Backend/Frontend de la aplicación.

    Para ello entonces ingrese a su cuenta de GCP y dirijase a la siguiente pestaña:
    ![imagen_prueba2](https://i.gyazo.com/232ab14624e531e0061b4a8cf1bd2419.png)
    y cree una nueva zona con el nombre que desee, y en la dirección del DNS coloque el dominio que generó en freenom.com con un . al final.
    ![imagen_prueba3](https://i.gyazo.com/aa7ebbcb6ec022092de745a9e6c8f2db.png)
    incialmente tendrá por defecto los dos últimos conjuntos de registros por defecto y estos servirán para establecer conexión con el dominio y verificarlo.
    
    El que tiene letras raras es necesario para la creación del certificado SSL que pide ZeroSSL para autenticar el sitio.
    ![imagen_prueba4](https://i.gyazo.com/90f7686ba9b37096a2d5999af8015b49.png)
    ![imagen_prueba5](https://i.gyazo.com/faf3e5421fd5a5b24eea6835e100e614.png)
    > Importante que en este coloque el **"www"** en el nombre del DNS y el **"."** al final de la dirección en el nombre del dominio junto con el tipo CNAME.

    Finalmente, tendrá que crear este conjunto de registros para asociar la dirección de la máquiana(**¡dirección pública y preferiblemente estática!)** hacia la que redirige el dominio.
    ![imagen_prueba6](https://i.gyazo.com/7f9a28097602541b52c89b1faef679f9.png)
    > **Nota**: No se introducen puertos en esta dirección debido a que NGINX asocia los puertos según el dominio que tiene configurado en su servidor (esto lo haremos más abajo xd).

    Por último el dominio deberá usar los siguientes dominios que encuentra en uno de los conjuntos de registros que vienen por defecto al momento de crear la zona para ajustarlos en el dominio dentro de freenom 
    ![imagen_prueba7](https://i.gyazo.com/8053f7789ad815cc3236d316fb6c134b.png)
    
    Para ello tendrá que ubicarse en freenom en sus dominios 
    ![imagen_prueba8](https://i.gyazo.com/a029f7753fe15fce9acfcd9ea6ae2a7c.png)
    posteriormente darle click en manage domain, y luego en servidores de nombres.
    ![imagen_prueba9](https://i.gyazo.com/c400ef07eca1a0c24140531ed3cbed0e.png)
    para poder colocar los que sacó del CLOUD DNS de GCP
    ![imagen_prueba10](https://i.gyazo.com/7ba84c1dbd2a19d1f2e3749fe2ab55ca.png)
    Ya con esto el dominio quedará apuntando a la dirección IP deseada satisfactoriamente!
- Etiquetas de red máquinas GCP

    >las máquinas por defecto que están en GCP tienen unas etiquetas de red que solo permiten tráfico HTTP/S desde ciertas máquinas que tengan anclada dicha etiqueta, esto puede imposibilitar que ciertas peticiones sean enviadas por lo tanto se deben modificar de la siguiente manera:

    ![](https://i.gyazo.com/f1ba81d5018fa58118866d3d09214280.png) las etiquetas de red o reglas de firewall deberían permitir flujo desde cualquier puerto **(¡Ojo! debería ser el 443 nada más porque ahí es donde se conecta por HTTPS, pero así está sirviendo para HTTP/S para pruebas na mas, solo es para tenerlo como consideración.)** los filtros de fuente únicamente indican la dirección por la cuál puede ser accedida, que es relativo al lanzamiento de la máquina por lo tanto se ejecuta en localHost 0.0.0.0 .   


# Instrucciones

Ubiquese en la carpeta del proyecto Django que contenga el archivo manage.py e instale python con el siguiente comando:
```
sudo apt-get update
sudo apt-get install python3-pip -y
```

## Crear un entorno virtual con Virtual env

```
sudo apt-get update 
sudo apt-get install  python3-venv -y
mkdir -p ~/entornos_virtuales/env_avanzo
python3 -m venv ~/entornos_virtuales/env_avanzo
source ~/entornos_virtuales/env_avanzo/bin/activate
```
>Le llamaremos env_avanzo al entorno virtual python y lo guardaremos en una carpeta llamada entornos_virtuales.

## Instalar dependencias
>**NOTA**: asegurese de que el entorno virtual de python esté activo antes de incializar este proceso.

localice el archivo requirements.txt donde se ubican las dependencias y ejecute los siguientes comandos:

```
sudo apt-get install gcc
pip3 install -r requirements.txt
```

> Esto instalará las dependencias necesarias en el entorno virtual.

## Conexión con el proyecto Django
    cd nombre_proyecto
>**Importante** hacer una prueba con 
```python3 manage.py runserver 0.0.0.0:8000``` para confirmar que el proyecto está corriendo correctamente.

### Configuración archivo Settings.py

```python
...
# SECURITY WARNING: el debug tiene que ser cambiado a falso si quiere correrse el proyecto en modo producción, osea habilitar HTTPS
DEBUG = False
# Aquí ponemos el nombre del dominio del sitio o la dirección IP que se tenga para la redirección del servidor.
ALLOWED_HOSTS = ["avanzo.tk"]

# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
...
```

## Incialización de uWSGI
Será necesario generar un archivo.py con la siguiente información para testear que las peticiones están llegando correctamente. 
```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World!"]
```
posteriormente ejecute el siguiente comando:

    uwsgi --http :8000 --wsgi-file archivo.py

## Instalación de NGINX

    sudo apt-get install nginx

posteriormente a la instalación, NGINX estará corriendo en segundo plano y recibirá peticiones redirigiendo a una ventana similar a la siguiente:
![imagenNGINX](https://cdn.wp.nginx.com/wp-content/uploads/2014/01/welcome-screen-e1450116630667.png)

>**NOTA**: Puede realizar una petición de GET de prueba al navegador sin el puerto y será redirigido a este template.

Posterior a la creación es necesario generar un archivo de configuración para que el sistema reciba peticiones correctamente, y ajuste los certificados. Por lo tanto ejecute el siguiente comando para crear un archivo de configuración (el nombre es arbitrario).

    sudo nano /etc/nginx/sites-available/avanzo.conf

Ingrese la siguiente información dentro:
```conf
# the upstream component nginx needs to connect to
upstream django {
    server unix:///home/rbrina4/nombre_proyecto/nombre_proyecto.sock;
    #rbrina4 corresponde al user que se conecta a la máquina virtual en GCP, nombre_proyecto carpeta donde está el proyecto django
}
# Aquí se configura el servidor que recibirá las peticiones HTTPS y los certificados asociados al dominio 
## Servidor que redirecciona peticiones HTTP hacia HTTPS, esto fuerza a que todas las peticiones sean HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name avanzo.tk;
    return 301 https://avanzo.tk$request_uri;
}
server {
    #Puerto que se anclará a las direcciones HTTPS y ubicación del certificado con la llave privada
    listen      443 ssl;
    ssl_certificate /home/rbrina4/nombre_proyecto/certificados/certificate.crt;
    ssl_certificate_key /home/rbrina4/nombre_proyecto/certificados/private.key;
    #Aquí va la el nombre del servidor 
    server_name avanzo.tk;
    charset     utf-8;
    # max upload size
    client_max_body_size 75M;
    # Django media and static files, esto sirve para controlar archivos que son estáticos y corresponden a assets de la página.
    location /media  {
        alias /home/rbrina4/nombre_proyecto/media;
    }
    location /static {
        alias /home/rbrina4/nombre_proyecto/static;
    }
    # Send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /home/rbrina4/nombre_proyecto/uwsgi_params;
    }
}

```
>**IMPORTANTE**:Para este punto es necesario que cuente con los certificados de su dominio dentro de la carpeta del proyecto Django o en la máquina junto con la clave privada, esta la puede conseguir de forma gratuita en ZeroSSL al momento de de crear el dominio, el cuál puede también adquirir de forma gratuita en freenom.com, este último tiene maña ya que debe introducir dominios con terminación .tk o .ml para que pueda ser redirigido correctamente a la pestaña de checkout en donde deberá crear una cuenta para poder administrar el dominio.

posteriormente ejecute el siguiente comando para crear los permisos del servidor de aplicaciones

    sudo nano /home/rbrina4/nombre_proyecto/uwsgi_params
Luego ejecute el siguiente comando para copiar el arhivo de configuración en la carpeta de sitios activados.

    sudo ln -s /etc/nginx/sites-available/nombre_proyecto.conf /etc/nginx/sites-enabled/

> Es importante que corra el siguiente comando para no tener problemas con la sintaxis del archivo de configuración: ```sudo nginx -t -c /etc/nginx/nginx.conf``` esto le dirá cuál es el problema y en qué linea se encuentra este, en tal caso de que haya un error también es posible consultar el log que provee el error en su stacktrace.

# Manejo de directorios con información estática:
Añada la variable STATIC_URL y STATIC_ROOT junto con el import os dentro del archivo Settings.py del proyecto Django para el manejo de información estática. 
```python
# en settings.py
import os
from pathlib import Path
# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent
...
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

ubiquese en la carpeta del proyecto y ejecute ```python3 manage.py collectstatic```
Una vez terminada la configuración del servidor de aplicaciones NGINX reinicie el servicio de NGINX con el siguiente comando para aplicar las migraciones. 

    sudo /etc/init.d/nginx restart

### Test para archivos estáticos
    
realice los siguientes comandos al interior de la carpeta del proyecto Django. 

    mkdir media
    wget https://upload.wikimedia.org/wikipedia/commons/b/b9/First-google-logo.gif -O media/media.gif

Esto generara una imagen a la que generaremos una petición get para verificar que las imagenes son cargadas correctamente. Finalmente para hacer la prueba realice la siguiente petición en el navegador;

    https://avanzo.tk/media/media.gif

# uWSGI y NGINX juntos en acción

Ejecute el siguiente comando al interior de la carpeta del proyecto Django

    uwsgi --socket nombre_proyecto.sock --module nombre_proyecto.wsgi --chmod-socket=666

Esto hará que la aplicación se encuentre corriendo en modo de prueba pero recibiendo las peticiones HTTPS.

# Produccion!

si desea puede generar una configuración de producción para que su máquina esté recibiendo peticiones en segundo plano con el siguiente comando dentro de la carpeta del proyecto Django:

    sudo nano /nombre_proyecto/nombre_proyecto_uwsgi.ini

En donde seteará la siguiente información:
```conf
[uwsgi]
chdir            = /home/rbrina4/nombre_proyecto/
module           = nombre_proyecto.wsgi
home             = /home/rbrina4/entornos_virtuales/env_avanzo
master          = true
processes       = 10
socket          = /home/rbrina4/nombre_proyecto/nombre_proyecto.sock
chmod-socket    = 666
vacuum          = true
daemonize       = /home/rbrina4/uwsgi-emperor.log
```

el cuál podrá correr con el siguiente comando:

    uwsgi --ini nombre_proyecto_uwsgi.ini

## Felicitaciones

Se acabó xd.

## Consideraciones finales
ejecute el comando desde el siguiente comando dentro del proyecto Django, y para Thais.

    source entornos_virtuales/env_avanzo/bin/activate    
    uwsgi --http :8080 --wsgi-file avanzo/wsgi.py

    python3 -m pip install --upgrade pip
    python3 -m pip install --upgrade Pillow

**COMANDOS CHEVERES**
git pull && yes yes | python3 manage.py collectstatic && wsgi --http :8080 --wsgi-file avanzo/wsgi.py

git pull && git add . && git commit -m "update credito create and roles" && git push

*¡LAS MIGRACIONES!*

**PILLOW**
python3 -m pip install --upgrade pip
python3 -m pip install --upgrade Pillow

**Para matar los bichos del puerto**
sudo fuser 8000/tcp
sudo fuser -k 8000/tcp
