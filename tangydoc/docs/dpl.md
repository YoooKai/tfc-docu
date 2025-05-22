
# DPL

## Explicación detallada paso a paso del despliegue con Gunicorn, Supervisor, Nginx y Certbot en Ubuntu

Me he informado y he explorado opciones y maneras de despliegue. He elegido **digitalocean**.

En primer lugar, creé un Droplet tras registrarme, eligiendo un plan de acuerdo a mis necesidades. Elegí el más sencillo.

![image.png](assets/images/image%2043.png)

![image.png](assets/images/image%2044.png)

Y comprar un dominio, para lo que elegí namecheap, pues como el nombre indica, es económico. Lo conecté con la IP del droplet de digitalocean.

![image.png](assets/images/image%2045.png)

### 1. Conectarse al servidor por SSH

```bash
ssh root@178.128.35.47

```

- Te conectas al servidor remoto usando el usuario root.
- La IP es la del servidor en DigitalOcean (o similar).
- Se te pedirá la contraseña.

---

### 2. Actualizar paquetes del sistema

```bash
sudo apt-get update
sudo apt-get upgrade

```

- `update`: actualiza la lista de paquetes disponibles.
- `upgrade`: instala las actualizaciones disponibles para paquetes ya instalados.
- Mantiene el sistema seguro y con las últimas versiones.

---

### 3. Instalar software necesario para el proyecto

```bash
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx
sudo apt install certbot python3-certbot-nginx

```

- `python3-pip`: gestor de paquetes Python.
- `python3-dev`: herramientas para compilar paquetes Python con extensiones nativas.
- `libpq-dev`: librerías para PostgreSQL, necesarias para la conexión desde Python.
- `postgresql`, `postgresql-contrib`: base de datos PostgreSQL y herramientas adicionales.
- `nginx`: servidor web que actuará como proxy inverso.
- `certbot` y `python3-certbot-nginx`: para obtener y gestionar certificados SSL de Let’s Encrypt.

---

### 4. Crear base de datos y usuario en PostgreSQL

```bash
sudo -u postgres psql

```

- Accedemos al prompt de PostgreSQL con usuario administrador `postgres`.

Dentro de PostgreSQL:

```sql
CREATE DATABASE tangy;
CREATE USER tangyuser WITH PASSWORD 'tangypassword';
ALTER ROLE tangyuser SET client_encoding TO 'utf8';
GRANT ALL PRIVILEGES ON DATABASE tangy TO tangyuser;
\q

```

- Creamos la base de datos llamada `tangy` para el proyecto.
- Creamos un usuario `tangyuser` con contraseña para conectarse a la DB.
- Configuramos la codificación UTF-8 para manejar acentos y caracteres.
- Le damos todos los permisos sobre la base de datos.

---

### 5. Instalar herramientas para Python y entornos virtuales

```bash
sudo -H pip3 install --upgrade pip
sudo -H pip3 install --virtualenv pip
sudo apt install python3-venv python3-pip

```

- Actualizamos `pip` (gestor de paquetes Python).
- Instalamos virtualenv para crear entornos aislados.
- Instalamos el paquete `python3-venv` para crear entornos virtuales con `venv`.

---

### 6. Crear estructura de carpetas y usuarios para la app

```bash
mkdir -p /webapps/tangy
cd /webapps/tangy

sudo groupadd --system webapps
sudo useradd --system --gid webapps --shell /bin/bash --home /webapps/tangy tangy

```

- Creamos carpeta `/webapps/tangy` donde vivirá la app.
- Creamos grupo de sistema `webapps` para manejar permisos.
- Creamos usuario sistema `tangy` con ese grupo, con home en la carpeta anterior, para correr la app con menos privilegios que root (por seguridad).

---

### 7. Crear entorno virtual Python para el proyecto

```bash
python3 -m venv environment_3_8_2
source environment_3_8_2/bin/activate
pip install --upgrade pip

```

- Creamos entorno virtual con Python 3.8.2 (o el que uses).
- Activamos el entorno para usar sus paquetes aislados.
- Actualizamos `pip` dentro del entorno.

---

### 8. Preparar archivo con dependencias del proyecto

En tu entorno local (donde desarrollas):

```bash
pip freeze > req.txt

```

- Esto genera una lista de todas las librerías usadas con sus versiones.

Ejemplo de dependencias (tu archivo `req.txt`):

```
asgiref==3.8.1
certifi==2025.1.31
cffi==1.17.1
charset-normalizer==3.4.1
cryptography==44.0.2
defusedxml==0.7.1
Django==5.2
django-cors-headers==4.7.0
django-rest-framework==0.1.0
djangorestframework==3.16.0
djangorestframework_simplejwt==5.5.0
djoser==2.3.1
idna==3.10
oauthlib==3.2.2
pillow==11.2.1
pycparser==2.22
PyJWT==2.9.0
python3-openid==3.2.0
requests==2.32.3
requests-oauthlib==2.0.0
social-auth-app-django==5.4.3
social-auth-core==4.5.6
sqlparse==0.5.3
stripe==12.0.0
typing_extensions==4.13.2
tzdata==2025.2
urllib3==2.4.0

```

---

### 9. Subir dependencias al servidor y instalar

En servidor:

```bash
touch req.txt
vi req.txt
# Pegar el contenido del archivo req.txt aquí
:wq  # para guardar y salir de vi

pip install -r req.txt
pip install psycopg2-binary

```

- Creamos el archivo `req.txt` con todas las dependencias del proyecto.
- Instalamos todas las librerías.
- `psycopg2-binary` es el adaptador para PostgreSQL en Python.

---

### 10. Subir proyecto Django al servidor

En local:

```bash
cd ..
zip -r djackets_tangy.zip tangy_django
scp djackets_tangy.zip root@178.128.35.47:.

```

- Salimos de la carpeta del proyecto y comprimimos todo.
- Subimos el zip al servidor con `scp`.

En servidor:

```bash
apt install unzip
unzip djackets_tangy.zip
rm djackets_tangy.zip

ls -larth
chown -R tangy:webapps .

```

- Instalamos unzip para descomprimir el zip.
- Descomprimimos y borramos el zip para ahorrar espacio.
- Cambiamos permisos para que el usuario tangy y grupo webapps sean dueños de los archivos.

---

### 11. Configurar Django para producción

```bash
cd tangy_django/main

cp settings.py settingsprod.py
vi settingsprod.py

```

- Creamos archivo `settingsprod.py` para la configuración de producción.

Dentro de `settingsprod.py`, configurar base de datos:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'tangy',
        'USER': 'tangyuser',
        'PASSWORD': 'tangypassword',
        'HOST': 'localhost',
        'PORT': '',
    }
}

```

- Configuras el acceso a la base de datos que creaste antes.

Luego:

```bash
cd ..
cp manage.py manageprod.py
vi manageprod.py

```

- Modifica `manageprod.py` para que use `settingsprod.py` en lugar de `settings.py` (cambia el `DJANGO_SETTINGS_MODULE`).

---

### 12. Migrar la base de datos y permisos en PostgreSQL

```bash
python manageprod.py makemigrations

```

- Detecta cambios en los modelos Django y crea migraciones.

Luego, en PostgreSQL:

```bash
sudo -u postgres psql
\c tangy
ALTER SCHEMA public OWNER TO tangyuser;
GRANT ALL PRIVILEGES ON SCHEMA public TO tangyuser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO tangyuser;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO tangyuser;
\q

```

- Cambia el propietario del esquema para que el usuario tenga permisos completos.

Finalmente:

```bash
python manageprod.py migrate

```

- Aplica las migraciones, creando tablas en la base.

---

### 13. Instalar Gunicorn y preparar script de inicio

```bash
pip install gunicorn
cd ..
vi environment_3_8_2/bin/gunicorn_start

```

Escribir script `gunicorn_start`:

```bash
#!/bin/sh

NAME='tangy_django'
DJANGODIR=/webapps/tangy/tangy_django
SOCKFILE=/webapps/tangy/environment_3_8_2/run/gunicorn.sock
USER=tangy
GROUP=webapps
NUM_WORKERS=3
DJANGO_SETTINGS_MODULE=tangy_django.settingsprod
DJANGO_WSGI_MODULE=tangy_django.wsgi
TIMEOUT=120

cd $DJANGODIR
source ../environment_3_8_2/bin/activate
export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DJANGODIR:$PYTHONPATH

RUNDIR=$(dirname $SOCKFILE)
test -d $RUNDIR || mkdir -p $RUNDIR

exec ../environment_3_8_2/bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
--name $NAME \
--workers $NUM_WORKERS \
--timeout $TIMEOUT \
--user=$USER --group=$GROUP \
--bind=unix:$SOCKFILE \
--log-level=debug \
--log-file=-

```

- Este script inicia Gunicorn con la configuración correcta, usando socket Unix, usuario y grupo específicos, con 3 trabajadores.

Luego:

```bash
chmod +x environment_3_8_2/bin/gunicorn_start

```

- Hacemos el script ejecutable.

---

### 14. Instalar y configurar Supervisor para mantener Gunicorn vivo

```bash
apt install supervisor
cd /etc/supervisor/conf.d/
touch tangy.conf
vi tangy.conf

```

Escribir en `tangy.conf`:

```
[program:tangy_django]
command = /webapps/tangy/environment_3_8_2/bin/gunicorn_start
user = tangy
stdout_logfile = /webapps/tangy/environment_3_8_2/logs/supervisor.log
redirect_stderr = true
environment=LANG=en_US.UTF-8,LC_ALL=en_US.UTF-8

```

- Configuramos Supervisor para que controle el proceso Gunicorn y lo reinicie si falla.

Luego:

```bash
mkdir /webapps/tangy/environment_3_8_2/logs/

supervisorctl reread
supervisorctl update
supervisorctl status

```

- Recargamos Supervisor para que detecte la nueva configuración, la aplique y verifiquemos el estado.

---

### 15. Configurar Nginx para servir la app y hacer proxy a Gunicorn

```bash
cd /etc/nginx/sites-available
vi apitangy.tangerinemess.com

```

Poner esta configuración:

```bash
upstream tangy_app_server {
    server unix:/webapps/tangy/environment_3_8_2/run/gunicorn.sock fail_timeout=0;
}

server {
    listen 80;
    server_name tangy.tangerinemess.com;
    return 301 https://tangy.tangerinemess.com$request_uri;
}

server {
    listen 443 ssl;
    server_name tangy.tangerinemess.com;

    client_max_body_size 4G;

    access_log /webapps/tangy/environment_3_8_2/logs/nginx-django-access.log;
    error_log /webapps/tangy/environment_3_8_2/logs/nginx-django-error.log;

    ssl_certificate /etc/letsencrypt/live/tangy.tangerinemess.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tangy.tangerinemess.com/privkey.pem;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

    location /static/ {
        alias /webapps/tangy/environment_3_8_2/tangy_django/static/;
    }

    location /media/ {
        alias /webapps/tangy/tangy_django/media/;
    }

    location / {
        root /webapps/tangy/tangy_vue/dist;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://tangy_app_server;
    }
}

```

- Definimos upstream con el socket de Gunicorn.
- Redirigimos HTTP a HTTPS.
- Configuramos SSL con certificados de Let’s Encrypt.
- Sirve archivos estáticos y media directamente para optimizar.
- Sirve frontend Vue desde carpeta `/webapps/tangy/tangy_vue/dist`.
- Reenvía las peticiones que empiezan por `/api/` a Gunicorn/Django.

Luego:

```bash
cd ../sites-enabled/
ln -s ../sites-available/apitangy.tangerinemess.com .
ls -larth
service nginx restart

```

- Creamos enlace simbólico para activar la configuración.
- Reiniciamos nginx para aplicar cambios.

---

### 16. Comprar dominio y configurar SSL con Certbot

```bash
sudo certbot -d tangy.tangerinemess.com
service nginx restart

```

- Certbot obtiene certificado gratuito para tu dominio.
- Reinicia nginx para aplicar el certificado y habilitar HTTPS.

Con esto tenemos Django en producción, con base de datos PostgreSQL, servidor Gunicorn gestionado por Supervisor, y nginx como proxy inverso con SSL.

---

## **Configuración de seguridad y dominio en Django (`settingsprod.py`)**

**Archivo:** `tangy_django/main/settingsprod.py`

### Añadir dominios al proyecto:

```python
ALLOWED_HOSTS = [
    "tangy.tangerinemess.com",
    "apitangy.tangerinemess.com",
]

CORS_ALLOWED_ORIGINS = [
    "https://tangy.tangerinemess.com",
    "https://apitangy.tangerinemess.com",
]

```

---

### **Reiniciar Gunicorn vía Supervisor**

```bash
supervisorctl restart tangy_django

```

---

### **Crear superusuario de Django**

```bash
source ../environment_3_8_2/bin/activate
python manageprod.py createsuperuser

```

Sigue los pasos para configurar un usuario administrador.

---

### **Cambiar URL hardcodeada en modelos**

**Archivo:** `product/models.py`

Reemplaza cualquier IP local (como `http://127.0.0.1:8000`) por:

```python
"https://apitangy.tangerinemess.com"

```

Después:

```bash
supervisorctl restart tangy_django

```

---

## **Despliegue del Frontend Vue**

### **Configurar el endpoint del backend en Vue**

**Archivo:** `src/main.js`

```
axios.defaults.baseURL = "https://apitangy.tangerinemess.com";

```

### **Compilar y empaquetar Vue**

```bash
npm run build
zip -r dist.zip dist

```

### **Enviar el build al servidor remoto**

```bash
scp dist.zip root@178.128.35.47

```

### **Desempaquetar en el servidor**

```bash
cp /root/dist.zip .
unzip dist.zip

mkdir tangy_vue
mv dist tangy_vue/dist

chown -R tangy:webapps .

```

---

## **Configuración del servidor NGINX para el Frontend**

### **Crear archivo de configuración NGINX**

```bash
cd /etc/nginx/sites-available
touch tangy.tangerinemess.com
vi tangy.tangerinemess.com

```

**Contenido:**

```
server {
    listen 80;
    server_name tangy.tangerinemess.com;
    return 301 https://tangy.tangerinemess.com$request_uri;
}

server {
    listen 443 ssl;
    server_name tangy.tangerinemess.com;

    client_max_body_size 4G;

    error_log  /webapps/tangy/environment_3_8_2/logs/nginx-vue-error.log;
    access_log /webapps/tangy/environment_3_8_2/logs/nginx-vue-access.log;

    ssl_certificate /etc/letsencrypt/live/tangy.tangerinemess.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tangy.tangerinemess.com/privkey.pem;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

    charset utf-8;
    root /webapps/tangy/tangy_vue/dist;
    index index.html index.htm;

    location / {
        root /webapps/tangy/tangy_vue/dist;
        try_files $uri /index.html;
    }
}

```

### **Activar el sitio en NGINX**

```bash
cd /etc/nginx/sites-enabled
ln -s ../sites-available/tangy.tangerinemess.com .

```

---

### **Reiniciar NGINX para aplicar cambios**

```bash
supervisorctl restart nginx

```

---

## **Verificación final**

- Backend en: [**https://apitangy.tangerinemess.com**](https://apitangy.tangerinemess.com/)
- Frontend en: [**https://tangy.tangerinemess.com**](https://tangy.tangerinemess.com/)

Abrir las URLs en el navegador y verifica que:

- Vue renderiza correctamente el frontend.
- Las llamadas a la API funcionan sin error de CORS o redirección.
- Django Admin está accesible vía `https://apitangy.tangerinemess.com/admin/`.

---