# Forwarding

Si queremos hacer DNAT, necesitamos: 

1. En primer lugar, editar el fichero `/etc/sysctl.conf` y descomentar la opción `net.ipv4.ip_forward=1` para habilitar el *forwarding* sobre IPv4.
2. Aplicar los cambios:
    ```
    sysctl -p
    ```
3. Ejecutar el siguiente comando de `iptables` (ajustando dirección IP, protocolo y puertos): 

    ```
    iptables -t nat -I PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 0.0.0.0:22
    ``` 