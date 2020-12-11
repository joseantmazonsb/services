# Servidor

## Instalación

1. Instalar wireguard: `apt update && apt install wireguard`.
2. Personalizar el fichero `docker-compose.yml` según las necesidades.
3. Ejecutar `docker-compose up -d` para iniciar el contendor.

## Posibles incidencias

### El contenedor no arranca al iniciar

Si modificas el fichero `docker-compose.yml`, asegúrate de que dejas `restart: always` y no le asignas otro valor, **ni siquiera `unless-stopped`**.

# Cliente

## Instalación

1. Instalar wireguard: `apt update && apt install wireguard`.
2. Navegar hasta la página web de vpn de nuestro servidor, hacer login, añadir nuestro cliente y descargar el fichero de configuración generado.
3. Poner el fichero de configuració descargado dentro de la carpeat `/etc/wireguard`.

## Posibles incidencias

### La conexión no es persistente

Es posible que te ocurra lo siguiente: inicias la conexión a la VPN, todo va bien, pero al rato ya no conecta, y la única forma de que lo haga es reiniciando o el cliente o el servidor usando `wg-quick`. Si esto sucede, probablemente estás detrás de un NAT, con lo necesitas añadir `PersistentKeepalive = 25` en la sección `[Peer]` del fichero de configuración de tu interfaz de Wireguard (`wg0.conf`, por ejemplo). Con esto hacemos que se envíen mensajes *keep alive* cada 25 segundos, de forma que no se cierra la conexión TCP.

### El servicio falla al iniciarse en boot time

La causa más probable es que `wg-quick` ya esté en marcha, con lo que vamos a hacer que el servicio se cargue este proceso cuando falle.
La ruta al fichero del servicio es probablemente `/etc/systemd/system/multi-user.target.wants/wg-quick@wgX.service`, donde `X` será el número indicado en el fichero de configuración de Wireguard; si el fichero se llama `wg0.conf`, el servicio será `wg-quick@wg0.service`.
No obstante, si no encontramos el fichero de servicio, siempre podemos usar `find`:
```
find / -name wg0.service
```
Una vez tenemos el fichero del servicio localizado, hay que añadir la línea `ExecStopPost=/usr/bin/wg-quick down %i` en la sección `[Services]` y ejecutar el comando `systemctl daemon-reload` para actualizar el servicio. Así, la próxima vez que lo iniciemos hará `wg-quick down` en caso de fallo, asegurando que es `systemd` quien se encarga de manejar la conexión.

