# Parcial 3 - Sistemas Distribuidos
## Nombre: Laura Aragón
## Código: A00268532
## Perfil Github: https://github.com/LauraAragon

### Objetivo
Realizar de forma autónoma el aprovisionamiento automático de infraestructura buscando lograr diagnosticar y ejecutar de forma autónoma las acciones necesarias para lograr infraestructuras estables.

### Descripción
Se aprovicionará un ambiente de contenedores con docker-compose, el cual contará con los servicios de: web-service, load-balancer, registrator y consul-server. El ambiente incluirá un contenedor que correrá el servicio de Discovery Service con Consul, en el cual se registrarán todos los servicios que surjan. La labor de registrar los servicios presentes es ejecutada por un Registrator. En el archivo de configuración .yml se tendrá una configuración base para un servidor web y una para el balanceador de cargas. 

### Desarrollo 
Para correr la solución se debe ejecutar el siguiente comando:

```bash
 $ docker-compose up --build -d
```

Una vez ejecutado el comando y gracias a la herramienta de administración de contenedores Portainer, podemos visualizar qué contenedores se han levantado. 

![][1]

Ahora, gracias al contenedor que corre el servicio de Registrator, todo servicio que se levante será anotado en el servidor de Consul Discovery Services como se muestra a continuación: 

![][4]   

Para escalar el web-server se debe tener en cuenta que en el archivo de configuración .yml en la parte relativa a este servicio se debe omitir el campo "container_name:". El comando para escalar el servico web es el siguiente:

```bash
$ docker-compose scale web=4
``` 

El escalamiento de los servidores web se realizará de forma manual, pero la configuración del load-balancer será automática gracias a la herramienta consul-template. Esta herramienta permite "recargar" la configuración del balanceador cada que un web-server es levantado. 

La plantilla de consul-template para realizar la configuración automática del balanceador de cargas Haproxy se encuentra en la carperta files. Además se cuenta con una configuración ya propia del consul-template que se presenta a continuación:

```json

template {
  source = "/tmp/haproxy.ctmpl"
  destination = "/etc/haproxy/haproxy.cfg"
  command = "haproxy -f /etc/haproxy/haproxy.cfg -sf $(pidof haproxy) &"
}

```

Lo que hace el servicio de consul-template es que, cada vez que se presente un cambio en el servido de descubrimiento de servicio Consul, este reconfigurará el load-balancer con la nueva dirección que ahora ofrece servicio web o al contrario eliminandola. 

Ahora se puede consultar el servicio web accediento al balanceador de cargas y este balanceará entre todas las direcciones que ofrezcan este servicio gracias al descubrimiento de servicios. 

![][3]

[1]: imagenes/containersList.png
[2]: imagenes/containerList2.png
[3]: imagenes/capturaWeb.png
[4]: imagenes/consulServices.png

### Referencias
https://livewyer.io/blog/2015/02/05/service-discovery-docker-containers-using-consul-and-registrator/
https://picodotdev.github.io/blog-bitix/2017/01/registro-y-descubrimiento-de-servicios-con-spring-cloud-y-consul/
http://microservices.io/patterns/microservices.html
http://sirile.github.io/2015/05/18/using-haproxy-and-consul-for-dynamic-service-discovery-on-docker.html
https://github.com/hashicorp/consul-template/tree/master/examples
https://portainer.io/