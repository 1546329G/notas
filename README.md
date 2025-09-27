# Guía Completa de Configuración de Servidor VPS con Nginx y Certificado SSL

## 1. Acceso Inicial al VPS

1. Abrir la terminal en tu PC.

2. Conectarse al servidor mediante **SSH**:

   ```bash
   ssh root@161.132.47.234
   ```

   > Aquí `root` es el usuario administrador y `161.132.47.234` es la IP pública del VPS.

3. **Actualizar el sistema** para contar con los últimos parches de seguridad:

   ```bash
   apt update && apt upgrade -y
   ```

4. Si aparece conflicto en algún archivo como `sshd_config`, elegir la opción de **mantener la versión local** para no perder acceso remoto.

---

## 2. Instalación del Stack LEMP

El **stack LEMP** (Linux, Nginx, MariaDB, PHP) permite servir sitios web y APIs dinámicas.

1. **Servidor Web (Nginx):**

   ```bash
   apt install nginx -y
   ```

   * Verificar instalación:

     ```bash
     systemctl status nginx
     ```

2. **Base de Datos (MariaDB):**

   ```bash
   apt install mariadb-server -y
   ```

   * Ejecutar configuración segura:

     ```bash
     mysql_secure_installation
     ```

     Aquí se define contraseña de root y se deshabilitan configuraciones inseguras.

3. **PHP-FPM (Procesador de PHP):**

   ```bash
   apt install php-fpm php-mysql -y
   ```

   Esto permite ejecutar aplicaciones dinámicas y conectar con MariaDB/MySQL.

---

## 3. Configuración de DNS y Subdominio

1. Ir al panel de **Hostinger**.
2. Crear un **registro A** para el subdominio:

   * Nombre: `elastica.pruebasjs.site`
   * Valor: `161.132.47.234` (IP del VPS).
3. Validar propagación de DNS:

   ```bash
   dig @8.8.8.8 elastica.pruebasjs.site
   ```

   * Si devuelve **NXDOMAIN**, es porque aún no se propagó.
   * Solución: eliminar y recrear el registro A para forzar propagación.

---

## 4. Configuración de Virtual Host en Nginx

1. Abrir el archivo de configuración:

   ```bash
   sudo nano /etc/nginx/sites-enabled/default
   ```
2. Reemplazar todo el contenido con:

   ```nginx
   server {
       listen 80;
       server_name elastica.pruebasjs.site;

       root /var/www/html;
       index index.php index.html index.htm index.nginx-debian.html;

       location / {
           try_files $uri $uri/ =404;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```
3. Verificar sintaxis de Nginx:

   ```bash
   nginx -t
   ```
4. Recargar servicio:

   ```bash
   systemctl reload nginx
   ```

---

## 5. Instalación de Certificado SSL con Certbot

1. Instalar Certbot y el plugin de Nginx:

   ```bash
   apt install certbot python3-certbot-nginx -y
   ```

2. Ejecutar la instalación del certificado:

   ```bash
   sudo certbot --nginx -d elastica.pruebasjs.site
   ```

3. Confirmar:

   * Certbot configurará automáticamente Nginx.
   * HTTPS quedará habilitado con redirección desde HTTP.

4. Verificar estado de certificados:

   ```bash
   sudo certbot certificates
   ```

5. Renovación automática (ya programada por defecto con systemd):

   ```bash
   sudo certbot renew --dry-run
   ```

---

## 6. Validaciones Finales

* Acceder en navegador:

  ```
  https://elastica.pruebasjs.site
  ```
* Verificar **candado verde** en el navegador.
* Comprobar logs de Nginx:

  ```bash
  tail -f /var/log/nginx/access.log
  tail -f /var/log/nginx/error.log
  ```

---

## 7. Buenas Prácticas y Recomendaciones

* **Usuarios y seguridad**:

  * Crear un usuario distinto de root para tareas diarias.
  * Configurar `ufw` (firewall) para permitir solo puertos 22 (SSH), 80 (HTTP), 443 (HTTPS).
* **Backups**: Mantener copia de `/etc/nginx/` y de las bases de datos MariaDB.
* **Monitoreo**: Instalar herramientas como `htop`, `vnstat`, `fail2ban`.
* **Escalabilidad**:

  * Colocar aplicaciones en `/var/www/mis_apps/`.
  * Usar `pm2` o `systemd` si luego se van a correr APIs en Node.js.

---

## 8. Estado Final

* El VPS con Ubuntu 20.04 quedó con **Nginx + SSL activo**.
* El subdominio `elastica.pruebasjs.site` apunta correctamente a la IP del servidor.
* El sitio responde por **HTTPS** con certificado válido de Let's Encrypt.
* Servidor listo para desplegar aplicaciones y APIs seguras.
 
