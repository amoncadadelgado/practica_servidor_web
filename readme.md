# Práctica servidor web

## Elementos importantes del docker-compose.yml

## Comando para crear una red
- Lo primero, es crear una red, en la cual se conectaran los futuros contenedores, para ello, entroducir en una terminal: 
    - docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 br02
    - Donde br02 (nombre de la red), y las ip el rango donde pertenece la red.

## Comandos para añadir los contenedores a una red.
En cada uno de los contenedores, luego de declarar la imagen, colocamos:

~~~
networks:
    br02:
        ipv4_address: 10.1.0.4
~~~

El comando anterior, indica que el contenedor pertenecera a la red br02 con la ip especificada como fija.

Una vez tenemos el comando anterior en cada uno de los contenedores, escribimos el siguiente comando a la altura de la etiqueta service:
~~~
networks: 
    br02:
        external: true
~~~
Dicho comando indica que la red br02 ya existe, y que se va a trabajar con ella. Lo cual evita que se cree una nueva red de forma predeterminada.

## Comando para asignar un servidor predeterminado al contenedor cliente.
Añadiendo el siguiente comando, a la altura de la etiqueta image de cada contenedor, se especifica que dicha ip hace referencía al servidor dns que atendera nuestras peticiones de manera predeterminada.
~~~
dns:
    - 10.1.0.4
~~~

En un principio funciona, pero hay que tener en cuenta que a partir de determinada versión del sistema operativo ya no se utiliza el archivo **resolv.conf** como configurador del servidor dns predeterminado. En cambio se utiliza el servicio systemd, el cual trabaja de otra forma. Por eso, cada vez que se reiniciaba el cliente, el valor predeterminado del archivo resolv.conf volvía al predeterminado (127.0.0.11).

## Procedimiento de creacion de los contenedores.
- En una terminal colocada en la ruta del archivo docker-compose.yml, escribimos el siguiente comando para crearlos:
    - docker-compose up

- Para iniciar el conjunto de contenedores especificados en el docker-compose.yml:
    - docker-compose start

- Para parar el conjunto de contenedores especificados en el docker-compose.yml:
    - docker-compose stop

- Para borrar el conjunto de contenedores especificados en el docker-compose.yml:
    - docker-compose down

- Para reinstalar el conjunto de contenedores especificados en el docker-compose.yml:
    - docker-compose restart

## Modificación de la configuración, arranque y parada de servicio bind9
Hay que especificar que cada vez que se realizan cambios en los archivos de configuración del contenedor, es obligatorio el reinicio del servicio, en nuestro caso basta con reiniciar el contenedor del servidor.

- Para parar el contenedor del servidor, basta con escribir el siguiente comando en una terminal:
    - docker **restart** nombre_del_contenedor

- Para parar el contenedor del servidor, basta con escribir el siguiente comando en una terminal:
    - docker **stop** nombre_del_contenedor

- Para iniciar el contenedor del servidor, basta con escribir el siguiente comando en una terminal:
    - docker **start** nombre_del_contenedor

- Para borrar el contenedor del servidor, basta con escribir el siguiente comando en una terminal:
    - docker **rm** nombre_del_contenedor