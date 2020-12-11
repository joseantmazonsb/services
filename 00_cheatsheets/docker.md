# Comandos básicos

# Resolviendo incidencias

## El contenedor no arranca al reiniciar el servidor

Si modificas un fichero `docker-compose.yml`, asegúrate de que pones `restart: unless-stopped` o `restart: always` si quieres que se inicie al arrancar el equipo.
