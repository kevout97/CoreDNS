# Plugins en CoreDNS

Entre las utilidades mas destacadas de CoreDNS encontramos la implementación de una serie de plugins que permiten ampliar enormemente la funcionalidad de CoreDNS.

## auto

El complemento **auto** se utiliza para un servidor DNS de "estilo antiguo". 

**Sintáxis**

```conf
auto [ZONES...] {
    directory DIR [REGEXP ORIGIN_TEMPLATE [TIMEOUT]]
    reload DURATION
    no_reload
    upstream [ADDRESS...]
}
```

Donde:

* **directory**: 
    * Carga zonas desde el **DIR** especificado . Si un nombre de archivo coincide con **REGEXP** , se utilizará para extraer el origen. 
    * **ORIGIN_TEMPLATE** se utilizará como plantilla para el origen. Las cadenas como {<number>}se reemplazan con las respectivas coincidencias en el nombre del archivo, por ejemplo, {1} es la primera coincidencia, {2} es la segunda. El valor predeterminado es: es db\.(.*) {1} decir, desde un archivo con el nombre db.example.com, el origen extraído será example.com. 
    * **TIMEOUT** especifica la frecuencia con la que CoreDNS debe escanear el directorio; el valor predeterminado es cada 60 segundos.

* **reload** Intervalo para realizar la recarga de la zona si la versión SOA cambia. El valor predeterminado es un minuto. Valor de los 0medios para no buscar cambios y recargar. p.ej. 30s comprueba el archivo de zona cada 30 segundos y vuelve a cargar la zona cuando la serie cambia.

* **no_reload** obsoleto. Establece la recarga a 0.

* **upstream** Define los resolutores en sentido ascendente que se utilizarán para resolver los nombres externos encontrados (como CNAME) que apuntan a nombres externos. 
    * **ADRESS** puede ser una dirección IP, un puerto IP: o una cadena que apunta a un archivo que está estructurado como /etc/resolv.conf. Si no se da una ADRESS , CoreDNS resolverá los CNAME contra sí mismo.

## bind

Normalmente, el oyente se enlaza al host comodín. Sin embargo, es posible que desee que el oyente se enlace a otra IP en su lugar.

**Sintáxis**

```conf
bind ADDRESS  ...
```

Donde:
* **ADDRESS** es una dirección IP para enlazar. Cuando se proporcionan varias direcciones, se abrirá un escucha en cada una de las direcciones.

## cache

Con la memoria caché habilitada, todos los registros, excepto las transferencias de zona y los registros de metadatos, se almacenarán en caché hasta 3600s.

**Sintáxis**

```conf
cache [TTL] [ZONES...]
```

Donde:

* **TTL**: Tiempo maximo de almacenamiento en segundos. Si no se especifica, se utilizará el TTL máximo, que es 3600 para las respuestas de error y 1800 para las negativas de existencia.
* **ZONES**: Si está vacío, se utilizan las zonas del bloque de configuración.

## dnssec

DNSSEC (Domain Name System Security Extensions) añade una capa de seguridad adicional a los servidores DNS de un dominio. Gracias a ello se previenen una gran cantidad de posibles actividades maliciosas.
Al utilizar DNSSEC se añaden firmas digitales en cada una de las partes implicadas: dominio, servidor DNS y Registry.

El funcionamiento, al acceder a un sitio con DNSSEC habilitado sería el siguiente:
* El navegador del visitante comprueba los servidores DNS asociados al dominio.
* Si las firmas digitales públicas que recibe coinciden con las publicadas en el Registry, el navegador dará por válida la solicitud y resolverá el sitio web, mostrando su contenido.
* Si por alguna razón las firmas no coinciden, el sitio web no sería accesible.

**Sintáxis**

```conf
dnssec [ZONES... ] {
    key file KEY...
    cache_capacity CAPACITY
}
```

Donde:

* **ZONES**: Se especifican las zonas que deben firmarse. Si está vacío, se utilizan las zonas del bloque de configuración.
* **key file**: Indica que los archivos KEY deben leerse desde el disco. Cuando se especifican varias claves, los conjuntos de recursos se firmarán con todas las claves. Generar una clave se puede hacer con dnssec-keygen: dnssec-keygen -a ECDSAP256SHA256 <zonename>. Una clave creada para la zona A puede ser utilizado con seguridad para la zona B . El nombre del archivo de clave se puede especificar en uno de los siguientes formatos:
    * nombre base de la clave generada Kexample.org+013+45330
    * clave pública generada Kexample.org+013+45330.key
    * clave privada generada Kexample.org+013+45330.private
* **cache_capacity**: Indica la capacidad de la caché. El complemento dnssec usa un caché para almacenar RRSIGs. El valor predeterminado para CAPACIDAD es 10000.

## errors

Cualquier error encontrado durante el procesamiento de la consulta se imprimirá en la salida estándar. Los errores de un tipo particular se pueden consolidar e imprimir una vez por un período de tiempo.

Este complemento solo se puede utilizar una vez por Server Block.

**Sintáxis**

```conf
errors
```

## file

El complemento de **file** se utiliza para un servidor DNS de "estilo antiguo". Si el archivo de zona contiene firmas (es decir, se firma usando DNSSEC), se devuelven las respuestas DNSSEC correctas. Sólo se admite NSEC! Si usa esta configuración, usted es responsable de volver a firmar el archivo de zona.

**Sintáxis**

```conf
file DBFILE [ZONES...]
```

Donde:

* **DBFILE**: Archivo de base de datos para leer y analizar. Si la ruta es relativa, la ruta de la directiva raíz será precedida.
* **ZONES**: Si está vacío, se utilizan las zonas del bloque de configuración.

## forward

Redirige la peticion al servidor destino. Facilita el envío de mensajes DNS a los solucionadores ascendentes.

**Sintáxis**

```conf
forward FROM ADDRES
```

Donde:

* **FROM** es el dominio base que debe coincidir para que se reenvíe la solicitud.
* **ADDRES** son los puntos finales de destino para reenviar. La sintaxis permite especificar un protocolo, tls://9.9.9.9o dns://(o ningún protocolo) para DNS simple. El número de upstreams está limitado a 15.

Este plugin permite añadir exepciones de dominios, para evitar el direcccionamiento de todos las peticiones, de tal forma que la sintáxis quedaria de la siguiente forma:

```conf
forward FROM ADDRES {
    except DOMAINS
}
```

Donde:

* **DOMAINS**: Es una lista de dominios separados por espacios para excluir del reenvío. Las solicitudes que no coincidan con ninguno de estos nombres se pasarán.

