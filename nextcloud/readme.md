# Instalación

1. Crear una carpeta `secrets` para los datos sensibles (contraseñas y otros datos) y crear un fichero txt por cada secreto indicado en `docker-compose.yaml`. Estos ficheros deben constar, simplemente, de una línea que contenga el valor del secreto. Por ejemplo, `secrets/mysql_root_pwd.txt` contendrá, en la primera línea, la contraseña del usuario `root` de MySQL.
2. Iniciar el contenedor: 
    ```
    docker-compose up -d
    ```