# Comandos básicos

| Comando | Utilidad |
|-|-|
| `docker ps`| Lista los contenedores activos. Con la opción `-a` también muestra los inactivos.
| `docker start/stop/restart` | Self explanatory.
| `docker login` | Inicia sesión en Docker Hub. Si se ejecuta sin argumentos pedirá introducir usuario y contraseña. |
| `docker system prune` | Elimina todos los contenedores intermedios. Con la opción `-a` también elimina todos los contenedores no usados, y con la opción `--volumes` también elimina todas las imágenes asociadas a dichos contenedores.

# Multiarch builds

Para obtener imágenes disponibles en distintas arquitecturas y subirlas a un repositorio de Docker Hub hay que seguir el siguiente proceso:

1. Habilitar las características experimentales de Docker. Para ello, debemos hacer login y añadir al principio del fichero `/root/.docker/config.json` lo siguiente:
    ```json
    "experimental": "enabled"
    ```
2. Instalar el plugin `buildx`. Seguir los pasos del [repositorio oficial](https://github.com/docker/buildx).
3. Ejecutar el contenedor multiarquitectura QEMU.
    ```
    docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
    ```
4. Crear un *builder* y seleccionarlo como *builder* a utilizar:
    ```
    docker buildx create --name builder-x64 --use
    ```
5. Habilitar los *targets* multiarquitectura:
    ```
    docker buildx inspect --bootstrap
    ```
6. Para cada servicio, entrar en su directorio raíz (donde se encuentre el fichero Dockerfile) y ejecutar:
    ```
    docker buildx build --platform <arch-1>,<arch-2>,<arch-n> --tag <docker-id>/<docker-repo>:<tag> --push .
    ```
    Por ejemplo, para construir una imagen para arquitectura AMD64 y ARM64, haríamos:
    ```
    docker buildx build --platform linux/amd64,linux/arm64 --tag joseantmazonsb/example-repo:latest --push .
    ```
# Resolviendo incidencias

## El contenedor no arranca al reiniciar el servidor

Si modificas un fichero `docker-compose.yml`, asegúrate de que pones `restart: unless-stopped` o `restart: always` si quieres que se inicie al arrancar el equipo.