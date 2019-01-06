# CoreDNS

![CoreDNS](https://www.cncf.io/wp-content/uploads/2018/02/coredns-horizontal-color.png)

**CoreDNS** es un servidor DNS escrito en **Go**, que a diferencia de otros servidores como BIND, Knot, PowerDNS y Unbound integra funcionalidades extra a traves de *plugins* precompilados.

Actualmente hay alrededor de 30 complementos incluidos en la instalación predeterminada de CoreDNS, pero también hay una gran cantidad de complementos externos que puede compilar en CoreDNS para ampliar su funcionalidad.

Entre los diferentes complementos integrados a CoreDNS podemos destacar aquellos que permiten implementar una comunicación con **Kubernetes**, leer datos de un archivo o bien de una base de datos, solo por mencionar algunos.