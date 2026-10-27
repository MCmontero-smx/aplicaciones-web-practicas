# Instalación de WordPress 

## 1. Conexión por SSH

Me conecté al servidor desde el equipo cliente con SSH:

```bash
ssh usuario@IP_DEL_SERVIDOR
```

En esta práctica, la dirección del servidor era `192.168.56.12`. SSH permite acceder a la terminal del servidor desde otro equipo.

![Conexión SSH al servidor](imagenes/imagen1.png)

## 2. Comprobación de Apache, MariaDB y PHP

Comprobé que Apache y MariaDB estuvieran activos y consulté la versión de PHP:

```bash
systemctl is-active apache2 mariadb
php -v
```

Los dos servicios deben indicar `active`. Apache sirve las páginas web, MariaDB almacena los datos y PHP ejecuta WordPress.

![Comprobación de Apache, MariaDB y PHP](imagenes/imagen2.png)

También revisé los sitios habilitados en Apache:

```bash
ls /etc/apache2/sites-enabled/
```

La actividad pide trabajar con un único sitio activo. Si se va a utilizar el sitio predeterminado, se pueden deshabilitar otros sitios usando sus nombres reales:

```bash
sudo a2dissite sitio1.conf sitio2.conf
sudo a2ensite 000-default.conf
sudo systemctl reload apache2
ls /etc/apache2/sites-enabled/
```

## 3. Creación de la base de datos y del usuario

Entré en MariaDB como administrador:

```bash
sudo mariadb
```

Creé la base de datos y el usuario para WordPress:

```sql
CREATE DATABASE wordpress;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'PON_AQUI_UNA_CLAVE_SEGURA';
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**¿Para qué sirven la base de datos y el usuario?** La base de datos guarda entradas, páginas, usuarios y ajustes. El usuario permite que WordPress acceda a esos datos sin utilizar la cuenta administradora de MariaDB.

![Creación de la base de datos y el usuario](imagenes/Captura%20de%202026-10-23%2009-20-43.png)

> Oculta la contraseña de la captura antes de subirla a GitHub.

## 4. Descarga e instalación de WordPress

Descargué WordPress, lo descomprimí y copié los archivos al directorio web de Apache:

```bash
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -a /tmp/wordpress/. /var/www/html/
```

El directorio `/var/www/html/` contiene los archivos que Apache publica en la web.

## 5. Permisos y archivo de configuración

Comprobé el directorio web y asigné sus archivos al usuario de Apache:

```bash
ls -ld /var/www/html
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```

![Comprobación del directorio web](imagenes/Captura%20de%202026-10-23%2009-35-45.png)

![Cambio de propietario y permisos](imagenes/Captura%20de%202026-10-23%2009-35-51.png)

Creé el archivo de configuración de WordPress a partir de la plantilla:

```bash
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sudo nano /var/www/html/wp-config.php
```

En el archivo configuré los datos de MariaDB:

```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wp_user' );
define( 'DB_PASSWORD', 'PON_AQUI_LA_CLAVE_DE_LA_BASE_DE_DATOS' );
define( 'DB_HOST', 'localhost' );
```

**¿Qué contiene `wp-config.php`?** Contiene la información necesaria para conectar WordPress con MariaDB. Como incluye una contraseña, no se debe subir a GitHub.

Comprobé la sintaxis PHP del archivo:

```bash
sudo php -l /var/www/html/wp-config.php
```

![Comprobación de sintaxis de wp-config.php](imagenes/Captura%20de%20pantalla%202026-10-27%20201542.png)

## 6. Instalación desde el navegador

Abrí la dirección del servidor:

```text
http://192.168.56.12/
```

WordPress mostró el asistente de instalación.

![Pantalla inicial del instalador](imagenes/Captura%20de%202026-10-23%2009-31-16.png)

En el formulario introduje el título del sitio y creé el usuario administrador con una contraseña segura.

![Formulario de instalación](imagenes/Captura%20de%202026-10-23%2009-45-09.png)

Al terminar, WordPress confirmó que la instalación se había completado.

![Instalación completada](imagenes/Captura%20de%202026-10-23%2009-45-27.png)

## 7. Comprobación del sitio y de la base de datos

Entré en el panel de administración:

```text
http://192.168.56.12/wp-admin/
```

![Escritorio de WordPress](imagenes/Captura%20de%202026-10-23%2009-45-58.png)

Abrí también el sitio público:

![Página pública de WordPress](imagenes/Captura%20de%202026-10-23%2009-46-33.png)

Creé una entrada y comprobé que se mostrara en la web:

![Editor de entradas](imagenes/Captura%20de%202026-10-23%2009-47-59.png)

![Entrada publicada](imagenes/Captura%20de%202026-10-23%2009-48-13.png)

Comprobé las tablas que WordPress creó en MariaDB:

```bash
sudo mariadb wordpress -e "SHOW TABLES;"
```

Tablas como `wp_posts`, `wp_users` y `wp_options` almacenan el contenido, los usuarios y los ajustes.

![Tablas de WordPress](imagenes/Captura%20de%202026-10-23%2009-50-48.png)

## 8. Dirección y ajustes del sitio

La dirección del sitio de la práctica es:

```text
http://192.168.56.12/
```

En `wp-config.php`, `WP_HOME` y `WP_SITEURL` especifican la dirección principal y la ubicación de WordPress. Deben coincidir con la dirección real del servidor:

```php
define( 'WP_HOME', 'http://' . $_SERVER['HTTP_HOST'] );
define( 'WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST'] );
```

**¿Qué puede pasar si la dirección está mal?** WordPress puede redirigir a una URL incorrecta o dejar de cargar correctamente.

![Configuración de WP_HOME y WP_SITEURL](imagenes/Captura%20de%20pantalla%202026-10-27%20201412.png)

En **Ajustes → Enlaces permanentes**, seleccioné **Nombre de la entrada** y guardé los cambios. Esta opción genera direcciones más legibles. Para que funcione, activé la reescritura de Apache:

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

![Ajustes de enlaces permanentes](imagenes/Captura%20de%20pantalla%202026-10-27%20202826.png)

Configuré el límite de subida de PHP en 64 MB:

```bash
printf 'upload_max_filesize = 64M\npost_max_size = 64M\n' | sudo tee /etc/php/8.3/apache2/conf.d/99-wordpress.ini
sudo systemctl restart apache2
```

`upload_max_filesize` limita el tamaño de cada archivo y `post_max_size` el tamaño total de la petición.

![Límites de subida de PHP](imagenes/Captura%20de%20pantalla%202026-10-27%20203207.png)

## 9. Error de conexión con MariaDB

Durante la actividad apareció el mensaje **Error establishing a database connection**. Significa que WordPress no pudo conectarse a MariaDB.

![Error de conexión con MariaDB](imagenes/Captura%20de%20pantalla%202026-10-27%20203619.png)

Para revisar el problema, comprobé que MariaDB estuviera activo:

```bash
systemctl is-active mariadb
```

Después comprobé que la base de datos tuviera las tablas de WordPress:

```bash
sudo mariadb wordpress -e "SHOW TABLES;"
```

También revisé que los valores de `DB_NAME`, `DB_USER`, `DB_PASSWORD` y `DB_HOST` en `wp-config.php` coincidieran con los configurados en MariaDB.

**¿Qué puede causar este error?** MariaDB puede estar detenido, los datos de conexión pueden ser incorrectos o el usuario puede no tener permisos. Para volver a conceder los permisos:

```sql
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
```

![Comprobaciones de los servicios y la base de datos](imagenes/Captura%20de%20pantalla%202026-10-27%20203902.png)

Tras revisar la configuración, volví a comprobar el sitio:

![Sitio WordPress tras las comprobaciones](imagenes/Captura%20de%20pantalla%202026-10-27%20203708.png)

## 10. Conclusiones y publicación en GitHub

La instalación quedó comprobada al poder entrar al panel de administración, ver el sitio público y consultar las tablas de WordPress en MariaDB.

Para que las imágenes aparezcan en GitHub, guardé todas las capturas en una carpeta llamada `imagenes`, junto al `README.md`. Añadí este `.gitignore` para evitar subir archivos privados:

```gitignore
wp-config.php
wp-config-*.php
*.sql
*.sql.gz
*.bak
*.backup
*.log
wp-content/uploads/
```

La documentación oficial de WordPress está disponible en [wordpress.org/documentation](https://wordpress.org/documentation/) y sus requisitos en [wordpress.org/about/requirements](https://wordpress.org/about/requirements/).
