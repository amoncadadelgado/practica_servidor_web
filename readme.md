# Práctica servidor web

## Elementos importantes del docker-compose.yml

### Creación del contenedor con el servidor apache:
~~~
apache_web:
    container_name: apache_server_practica
    image: httpd
    networks:
      br02:
        ipv4_address: 10.1.0.4
    ports:
      - 8008:8008
    volumes:
      - apache_index:/usr/local/apache2/htdocs
      - apache_conf:/usr/local/apache2/conf
~~~
- Asignamos una ip fija (10.1.0.4), dentro de la red br02 creada previamente.
- Asociamos los directorios, htdocs y conf a sendos volumenes, los cuales fueron creados previamente.

### Creación del contenedor del cliente:
~~~
apache_cliente:
    image: kasmweb/desktop:1.10.0-rolling
    networks:
      br02:
    stdin_open: true  # docker run -i
    tty: true         # docker run -t
    environment:
      VNC_PW: password
    ports:
      - 6901:6901
    dns:
      - 10.1.0.40
~~~
- El contenedor dispondra de una ip dinamica, dentro de la red br02 creada previamente.
- El servidor dns del cliente se encuentra en 10.1.0.40.
- Con el apartado environment, permitimos la conexión con el servidor, es muy importante con VNC_PW asignarle la password.

### Volumenes y conexiones de red:
~~~
volumes:
  apache_index:
    external: true
    name: apache-data-practica
  apache_conf:
    external: true
    name: apache-conf-practica
  conf:
    external: true
    name: dns-main_conf
networks: 
  br02:
    external: true
~~~
- Con el anterior codigo, indicamos que los volumenes asignados a los contenedores los busque en el exterior en vez de crearlos, y lo mismo acontece con la red utilizada para la conexión (br02).

### Archivos de configuración del dns:

- En el volumen conf, asociado a la ruta /etc/bind del contenedor, encontramos el archivo db.example.com con el siguiente codigo:
~~~
;
; BIND data file for example.com
;
$TTL	604800
@	IN	SOA	example.com. root.example.com. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	ns.example.com.
@	IN	A	10.1.0.4
@	IN	AAAA	::1
ns  IN  A   10.1.0.4
ggg	IN	A	10.1.0.4
maquina1	IN 	A 	10.1.0.4
ggg IN	TXT	"Aqui va un token de seguridad"
ooo	IN	CNAME	maquina1
~~~

- Como podemos comprobar, asiganamos las distintas direcciones registradas del dns a nuestro servidor apache, asignando la ip del servidor. 