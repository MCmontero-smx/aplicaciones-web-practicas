# Instalación y Securización de Apache

## Captura
Es una imagen despues de actualizar.

![Captura de la version actual de ubuntu.](Imagenes/Imagen1.png)

-----------------------------------
## Instalacion de apache2
![Captura de la instalacion de apache2.](Imagenes/Imagen2.png)

A la hora de instalar se instlaron paquetes adicionales como independecias: 
**libfwupd2 y libgusb2**
Apache: permite el tráfico básico necesario para que funcione el servidor web.

Apache Full: permite todo el tráfico de Apache, incluyendo conexiones HTTP y HTTPS.

Apache Secure: permite únicamente el tráfico HTTPS, que está cifrado y es más seguro.
----------------------------------
## Comprobación del funcionamiento

### Captura
Imagen del status de apache2

![Captura de status de apache2](Imagenes/Imagen3.png)

### Captura del navegador
![Captura web](Imagenes/Imagen4.png) 

-----------------------------------------------------------
 ## Diferencia de los perfiles de Apache
**Apache**: permite el tráfico básico necesario para que funcione el servidor web.

**Apache Full**: permite todo el tráfico de Apache, incluyendo conexiones HTTP y HTTPS.

**Apache Secure**: permite únicamente el tráfico HTTPS, que está cifrado y es más seguro.
## Apartado 4. Comandos principales de administración

| Comando | Función |
|---|---|
| `sudo systemctl start apache2` | Inicia el servidor Apache. |
| `sudo systemctl stop apache2` | Apaga el servicio Apache. |
| `sudo systemctl restart apache2` | Reinicia Apache para aplicar cambios. |
| `sudo systemctl reload apache2` | Actualiza la configuración sin reiniciar el servicio. |
| `sudo systemctl enable apache2` | Configura Apache para iniciarse al arrancar el sistema. |
| `sudo systemctl disable apache2` | Evita que Apache se inicie automáticamente. |
| `apache2ctl configtest` | Comprueba si la configuración tiene errores. |
| `apache2ctl -S` | Muestra los sitios web virtuales configurados. |
| `apache2ctl -M` | Muestra los módulos de Apache disponibles y cargados. |
| `a2enmod / a2dismod` | Habilita o deshabilita módulos de Apache. |
| `a2ensite / a2dissite` | Habilita o deshabilita sitios web configurados. |
| `a2enconf / a2disconf` | Habilita o deshabilita configuraciones adicionales. |

Conviene usar `reload` cuando modificas la configuración de Apache y quieres aplicarla sin interrumpir las conexiones activas. Usa `restart` cuando necesitas reiniciar completamente el servicio.

---------------------------------------
## Apartado 5. Ficheros y Practicadirectorios importantes

| Ruta | Descripción |
|---|---|
| `/etc/apache2/apache2.conf` | Archivo de configuración principal. |
| `/etc/apache2/ports.conf` | Define los puertos donde escucha Apache. |
| `/etc/apache2/sites-available/` | Contiene los sitios web configurados disponibles. |
| `/etc/apache2/sites-enabled/` | Contiene los sitios web activados. |
| `/etc/apache2/mods-available/` y `mods-enabled/` | Contienen los módulos disponibles y activados. |
| `/etc/apache2/conf-available/` y `conf-enabled/` | Contienen las configuraciones adicionales disponibles y activadas. |
| `/etc/apache2/envvars` | Define variables de entorno de Apache. |
| `/var/www/html/` | Directorio donde se guardan los archivos web por defecto. |
| `/var/log/apache2/access.log` | Guarda el registro de accesos al servidor. |
| `/var/log/apache2/error.log` | Guarda los errores del servidor. |
### Captura contenido /etc/apache2
![Captura del contenido del /etc/apache2/.](Imagenes/Imagen5.png)
### Pregunta: ¿Por qué Apache usa enlaces simbólicos entre los directorios *-available y *-enabled

Para poder activar o desactivar sitios, módulos y configuraciones fácilmente, sin mover ni duplicar los archivos originales. Los directorios `*-available` guardan los disponibles y `*-enabled` contiene enlaces a los que están activados.

--------------------------------------------------

## Apartado 6. Modificaciones típicas del servicio

En este apartado he configurado Apache para que escuche en el puerto `8080`, he creado los directorios de dos sitios web y he comprobado que el servidor responde.

### 6.1. Configurar Apache para escuchar en el puerto 8080

Primero he editado el archivo `/etc/apache2/ports.conf` para añadir el puerto `8080`:

```apache
Listen 8080
````
![Captura del contenido del /etc/apache2/.](Imagenes/Imagen7.png)

A continuación, he editado la configuración del sitio predeterminado para que su VirtualHost use ese puerto:
```Apache
<VirtualHost *:8080>
```

![Captura del contenido del /etc/apache2/.](Imagenes/imagen6.png)
### 6.3. Configurar el nombre del servidor

Para definir el nombre del servidor y evitar el aviso de Apache sobre el nombre de dominio completo, he creado el archivo servername.conf con la directiva ServerName localhost. Después, he habilitado la configuración y he recargado Apache:
```Apache
echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/servername.conf
sudo a2enconf servername
sudo systemctl reload apache2
```

![Captura del contenido del /etc/apache2/.](imagenes/imagen9.png)

### 6.4. Cambiar el correo del administrador 
En la parte inferior del archivo  se encuentra el cambio de correo algo como asi:
```Apache
ServerAdmin correo@admin.com
```
![Captura del contenido del /etc/apache2/.](imagenes/imagen102.png)

### 6.5. Personalizar una página de error 
En la parte superio del archivo se encuetra personalizado el error 404 el cambio como algo asi:
```Apache
ErrorDocument 404 /error4004.html
```
![Captura del contenido del /etc/apache2/.](imagenes/imagen101.png)

## Apartado 7. Módulos de Apache

| Módulo | Función |
|---|---|
| `mod_rewrite` | Reescritura de URL |
| `mod_headers` | Manipulación de cabeceras HTTP |
| `mod_ssl` | Soporte HTTPS |
| `mod_dir` | Gestión de ficheros índice (`index.html`) |
| `mod_alias` | Redirecciones y alias de rutas |
| `mod_auth_basic` | Autenticación básica |
| `mod_status` | Página de estado del servidor |
| `mod_php` / PHP-FPM | Ejecución de PHP (opcional) |

**Pregunta**
Apache puede mostrar un error de configuración o dejar de funcionar correctamente porque el módulo dependiente necesita el módulo desactivado.

------------------------
 ## Apartado 8. Varios sitios en un mismo servidor

En la imagen 11 se ve cómo creé dos directorios para alojar los sitios web y añadí una página de inicio a cada uno:
```Apache
sudo mkdir -p /var/www/sitio1 /var/www/sitio2
echo "<h1>Bienvenido a SITIO 1</h1>" | sudo tee /var/www/sitio1/index.html
echo "<h1>Bienvenido a SITIO 2</h1>" | sudo tee /var/www/sitio2/index.html
```
**Captura**

![Captura del contenido del /etc/apache2/.](Imagenes/Imagen13.png)

**Pregunta**
Los VirtualHosts basados en nombre distinguen los sitios por el dominio; los basados en IP, por la dirección IP. En este caso usé uno basado en IP (192.168.56.12).
------------------

## Apartado 9. Permisos y propietarios

Cambiamos a los permisos correspondientes como en la captura.

![Captura del contenido del /etc/apache2/.](Imagenes/Imagen12.png)

Y ahora quitamos los permisos del sitio1 para ver el error 403

![Captura del contenido del /etc/apache2/.](Imagenes/ultima.png)
