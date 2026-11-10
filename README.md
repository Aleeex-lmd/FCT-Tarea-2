# FCT-Tarea-2

## Copia de los correos que has enviado a los profesores (petición de nombres DNS y de certificado), los nombres DNS pedidos con la justificación de cada uno (interno/externo) y la comprobación con dig de que resuelven.

![alt text](img/0.png)


![alt text](img/1.png)

## PHP2Shell: versión de PHP instalada, resultado de tu comprobación del changelog / Debian Security Tracker y decisión tomada si no estaba corregida.

Me he decantado por la version 8.4.27-1~deb13u1 de php ya que corrije las dos vulnerabilidades de las versiones anteriores y es la version mas nueva de de php 8.4 por lo cual su instalacioon es bastante sencilla

![alt text](img/2.png)

Como podemos ver estan ambas vulnerabilidades resueltas por lo que no tenemos que hacer nada mas.

## server2:
    - Configuración de pg_hba.conf y de postgresql.conf que has modificado (sólo las líneas relevantes).
   ![alt text](img/3.png)
   ![alt text](img/4.png)

    - Salida de \conninfo y de pg_stat_ssl mostrando conexión cifrada, y capturas de rechazo sin TLS / desde otra IP.
   ![alt text](img/5.png)
   ![alt text](img/6.png)
   ![alt text](img/7.png)
    
    - Fichero /etc/nftables.conf, salida de sudo nft list ruleset y resultado de las pruebas con nmap/nc.

   ![alt text](img/8.png)

   ![alt text](img/9.png)
    
    - Configuración de Redis relevante (bind, autenticación).
   ![alt text](img/10.png)

## server1:

    - Fichero de configuración del pool de PHP-FPM y de PHP que has creado.
   ![alt text](img/11.png)

    - Server block de nginx, salida de nginx -t y comprobación del certificado (openssl s_client) y de las cabeceras (curl -I).

    ```conf
    alex@server1:~$ sudo cat /etc/nginx/sites-available/00-moodle.conf
server {
    if ($host = moodle-testing.gonzalonazareno.org) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name moodle-testing.gonzalonazareno.org 172.22.8.196;
    return 301 https://$host$request_uri;


}

server {
    listen 443 ssl default_server;
    http2 on;
    server_name moodle-testing.gonzalonazareno.org 172.22.8.196;

    root /var/www/moodle/public;
    index index.php index.html index.htm;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_certificate /etc/letsencrypt/live/moodle-testing.gonzalonazareno.org-0001/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/moodle-testing.gonzalonazareno.org-0001/privkey.pem; # managed by Certbot
    
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    server_tokens off;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    access_log /var/log/nginx/moodle_access.log;
    error_log /var/log/nginx/moodle_error.log;

    client_max_body_size 100M;

    location ^~ /moodledata/ {
        return 404;
    }

    location ~* /moodledata/.*\.(php|php5|phtml)$ {
        deny all;
    }

    location ~ /\.(?!well-known).* {
        return 404;
    }

    location ~* /(?:behat|tests|fixtures|node_modules|vendor)/ {

    ```
   ![alt text](img/12.png)

   ![alt text](img/13.png)

    - Estructura de directorios y permisos de Moodle, moodledata y config.php (ls -l, namei -l) (sin mostrar contraseñas).

   ![alt text](img/14.png)

    - Comando de instalación CLI utilizado (sin contraseñas) y captura de Moodle funcionando por HTTPS.

    ```bash
    sudo -u www-data php /var/www/moodle/public/admin/cli/install.php \
    --chmod=2770 \
    --lang=es \
    --wwwroot=https://moodle-testing.gonzalonazareno.org \
    --dataroot=/var/moodledata \
    --dbtype=pgsql \
    --dbhost=localhost \
    --dbname=moodle \
    --dbuser=moodleuser \
    --dbpass="***" \
    --prefix=mdl_ \
    --fullname="Moodle Testing Platform" \
    --shortname="MoodleTest" \
    --adminuser=admin \
    --adminpass="***" \
    --adminemail=admin@gonzalonazareno.org \
    --agree-license
    ```

    ![alt text](img/15.png)


    - Ficheros .service y .timer del cron y salida de systemctl list-timers.

    ![alt text](img/16.png)
    
    - Configuración de sesiones/caché de Moodle en Redis y salida de redis-cli con claves de sesión.

    ![alt text](img/17.png)
    ![alt text](img/18.png)
    
    - Filtro y jail de fail2ban para Moodle y prueba del baneo.

    ![alt text](img/19.png)
    
    - Prueba de que no se ejecuta PHP en directorios de subidas.

    ![alt text](img/20.png)
    
    
    - Fichero drop-in de systemd de PHP-FPM y salida de systemd-analyze security antes y después


Respuesta a las preguntas planteadas:

    ¿Por qué es importante que el código de Moodle no sea escribible por el usuario de PHP-FPM y que moodledata esté fuera de la raíz web?
    ¿Por qué un servidor de base de datos no debe tener registro en el DNS externo y cómo sirve el certificado comodín para cifrar la conexión con PostgreSQL?
    ¿Qué ventajas e inconvenientes tiene guardar las sesiones de Moodle en Redis?
    ¿En qué consiste la vulnerabilidad PHP2Shell y qué medidas has tomado (y cuáles no te habrían protegido)?
    ¿Qué diferencia hay entre verify-ca y verify-full en PostgreSQL?
