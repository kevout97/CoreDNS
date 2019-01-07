# Instalacion CoreDNS (Docker)

## Concimientos previos

Un servidor DNS (Domain Name System - Sistema de nombres de dominio) es un servidor que traduce nombres de dominio a IPs y viceversa. 

## Requisitos

* Docker 1.12.x o mayor
* Espacio en Disco Duro 50 GB.
* Memoria RAM 2GB
* Cores 2

*NOTA*: Las especificaciones son considerando que el servidor sera dedicado para el servicio DNS y algunos otros procesos que no consuman grandes recursos.

## Despliegue

Para llevar a cabo la implementación de la herramienta **CoreDNS** es necesario descargar la imagen desde los repositorios oficiales en DockerHub.

```bash
docker pull coredns/coredns
```

A continuación realizamos el despliegue del servidor a traves del comando:

```bash
docker run -d --name core_dns \
            -p 53:53/tcp \
            -p 53:53/udp \
            -v /var/containers/coredns/tmp:/tmp:z \
            coredns/coredns -conf /tmp/Corefile -dns.port 53
```

En este sentido es importante contar con el archivo **/var/containers/coredns/tmp/Corefile** (del lado del host) el cual contendrá toda la configuración de las zonas DNS. 
Un ejemplo de este archivo es el siguiente:

```conf
infra.ctin.com {
    file /tmp/infra.ctin.com
    errors
    log
}
```

En este ejemplo configuramos la zona **example.com** la cual vincula el archivo **/tmp/example.com** (path dentro del contenedor) el cual integra todos los registros correspondientes a esa zona.
A continuación se muestra el contenido de dicho archivo, ubicado en **/var/containers/coredns/tmp/example.com** (del lado del host):

```conf
$TTL    86400
@       IN      SOA     infra.ctin.com.  . (
                1267456417      ; Serial
                10800   ; Refresh
                3600    ; Retry
                3600    ; Expire
                3600)   ; Minimum
    IN          NS  dns1
    IN          NS  dns2
    IN          A   201.161.82.206


dns1            IN A        201.161.82.206
dns2            IN A        201.161.82.206
inicio          IN A        172.26.20.145
```

*La búsqueda del significado de cada registro se deja como ejercicio al lector.*

Para comprobar el funcionamiento del servidor ejecutamos el comando:

```bash
dig @localhost inicio.infra.ctin +short
```

Cuya salida nos mostrará la dirección que hemos configurado con anterioridad, para nuestro caso `172.26.20.145`.

**NOTA**:
* **-conf**: Esta bandera permite especificar la dirección del archivo de configuración principal, por convención, para *CoreDNS* este archivo se ubica en **/tmp** (dentro del contenedor) con el nombre de **Corefile**.
* **-dns.port** Esta bandera permite especificar el puerto por el que escuchará las peticiones el servidor DNS. En caso de no especificar dicho puerto, *CoreDNS* usa el ya conocido puerto **53**.

## Issues.

### rpc error: code = 2 desc = oci runtime error: exec failed: container_linux.go:247: starting container process caused "exec: \"bash\": executable file not found in $PATH"

Este error se suscita tras ejecutar el comando:

```bash
docker exec -it core_dns bash
```

Donde *core_dns* es el nombre del contendor.

Esto se debe a que la imagen esta desarrollada tomando como base dos imagenes, como primer capa tiene **debian:stable-slim** y encima la imagen **scratch** siendo esta la imagen a la cual accedemos, y cuyo unico contenido es el del proceso **coredns**.

### Error starting userland proxy: listen tcp 0.0.0.0:53: bind: address already in use.

Para corregir este error basta con cambiar los puertos usados por unos que no se encuentren ocupados, un ejemplo el puerto **1053**.