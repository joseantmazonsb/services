# Guía

## Redirección HTTPS

La redirección en Apache es muy simple, y en este caso la vamos a ver con un ejemplo. Si quisieramos redirigir todo el tráfico HTTP a nuestra web www.example.com, necesitamos crear dos virtual hosts, que pueden estar en el mismo fichero, de forma que el virtual host que escucha en el puerto 80 redirija al virtual host que se encuentra escuchando en el puerto 443 (y que habilita TLS):

```apache
<VirtualHost *:80>
	ServerName www.example.com
	ServerAlias www.example.com
	ServerAdmin hostmaster@example.com

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	RedirectPermanent / https://www.example.com
</VirtualHost>

<VirtualHost *:443>
	ServerName www.example.com
	ServerAlias www.example.com
	ServerAdmin hostmaster@example.com

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	DocumentRoot /var/www/example

	<Directory /var/www/example>
		Options Indexes FollowSymLinks MultiViews
		AllowOverride None
		Order allow,deny
		allow from all
	</Directory>

	SSLEngine on
	SSLCertificateFile /certs/fullchain
	SSLCertificateKeyFile /certs/privkey
</VirtualHost>
```

## Proxy inverso seguro

En el caso de que queramos dar un servicio a través de una máquina distinta a la que ejecuta el servidor Apache, pero queremos aprovechar nuestro servidor web, podemos montar un proxy inverso y dejar que este nos redirija a la máquina que realmente da el servicio, de forma que podemos reutilizar el canal seguro que ya nos ofrece el servidor web principal.

Para montar un proxy inverso sencillo solo necesitamos añadir lo siguiente al fichero de configuración del virtual host que hará de proxy:

```apache
	ProxyPreserveHost On
	ProxyPass / http://0.0.0.0:80/
	ProxyPassReverse / http://0.0.0.0:80/
```
*Nota: ajustar dirección IP y puerto según el caso de uso.*

Finalmente, es **MUY IMPORTANTE** que añadamos una barra (`/`) seguida del puerto para evitar posibles errores de redirección.

### Error al acceder a `/socket` u otras rutas

Si la aplicación a la que se está accediendo utiliza *websockets*,
es posible que tengas que especificar un par de directivas `ProxyPass` y `ProxyPassReverse` más, de la siguiente forma:

```apache
	ProxyPass "/socket" "ws://0.0.0.0:80/socket"
	ProxyPassReverse "/socket" "ws:http://0.0.0.0:80/socket"
```

### Autenticación básica

Si tenemos una página a la que se accede a través de un proxy inverso y no queremos que cualquiera pueda acceder a ella (porque está en construcción, contiene datos sensibles, es un entorno de pruebas, etc), podemos seguir los siguientes pasos para conseguirlo:

1. Crear un fichero de autenticación

    ```
    touch /etc/apache2/.htpasswd
    ```

2. Añadir un par usuario contraseña.

    ```
    htpasswd -c /etc/apache2/.htpasswd jose
    ```

3. Añadir lo siguiente al fichero de configuración del virtual host:

    ```apache
    <Proxy *>
        Order deny,allow
        Allow from all
        Authtype Basic
        Authname "Password Required"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Proxy>
    ```
