# Práctica servidor web

Elementos importantes del docker-compose.yml

## Creación del contenedor con el servidor apache:
~~~
apache_web:
    container_name: apache_server_practica
    image: httpd
    networks:
      br02:
        ipv4_address: 10.1.0.4
    ports:
      - 80:80
    volumes:
      - apache_index:/usr/local/apache2/htdocs
      - apache_conf:/usr/local/apache2/conf
~~~
- Asignamos una ip fija (10.1.0.4), dentro de la red br02 creada previamente.
- Asociamos los directorios, htdocs y conf a sendos volumenes, los cuales fueron creados previamente.

## Creación del contenedor del cliente:
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

## Creación del contenedor del wireshark:
~~~
wireshark:
    image: linuxserver/wireshark:latest
    container_name: wireshark
    cap_add:
      - NET_ADMIN
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - config_wireshark:/config
    ports:
      - 3000:3000 #optional
    restart: unless-stopped
~~~
Donde:
- **network_mode: host**, indica que el contenedor se conecta a la red del host.
- El resto de valores, es recomendable dejarlas tal cual se observan.


## Volumenes y conexiones de red:
~~~
volumes:
  apache_index:
    external: true
    name: apache-data-practica
  apache_conf:
    external: true
    name: apache-conf-practica1
  conf:
    external: true
    name: dns-main_conf
  config_wireshark:
    external: true
    name: wireshark_config
networks: 
  br02:
    external: true
~~~
- Con el anterior codigo, indicamos que los volumenes asignados a los contenedores los busque en el exterior en vez de crearlos, y lo mismo acontece con la red utilizada para la conexión (br02).

## Modificación del archivo de configuración del dns:
En el volumen conf_bind, asociado a la ruta /etc/bind del contenedor, encontramos el archivo db.example.com con el siguiente codigo:
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

; registro ns responsable de la zona.
@	IN	NS	ns.example.com.
ns  IN  A   10.1.0.40

; @	IN	A	10.1.0.4 Asocia el dominio example.com al servidor apache.
@	IN	A	10.1.0.4
@	IN	AAAA	::1

primera	IN	CNAME	example.com.
segunda	IN	CNAME	@
~~~
- Donde se añadieron dos CNAME, para que cada una de las direcciones apunte al servidor apache (10.1.0.4). Luego será el servidor apache el que decida que index devolver.
## Modificación de los archivos de configuración del apache:
En el volumen conf_apache, asociado a la ruta /usr/local/apache2/conf del contenedor, disponemos del archivo **httpd.conf** en el cual, descomentamos la línea del include:
~~~
# Virtual hosts
Include conf/extra/httpd-vhosts.conf
~~~
Una vez descomentada la anterior línea, entramos en el directorio **extra** abrimos el archivo **httpd-vhosts.conf** donde escribimos las siguientes líneas:
~~~
<VirtualHost primera.example.com:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/usr/local/apache2/htdocs/primera"
    ServerName example.com
    ServerAlias primera.example.com
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" common
</VirtualHost>

<VirtualHost segunda.example.com:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/usr/local/apache2/htdocs/segunda"
    ServerName example.com
    ServerAlias segunda.example.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>
~~~
Donde:
- **DocumentRoot:** asignamos la ruta de la carpeta que contiene el index correspondiente.
- **ServerName:** asignamos el nombre de dominio base.
- **ServerAlias:** especifica el nombre del subdominio.

Hay que resaltar que las rutas asignadas en DocumentRoot deben existir, para ello, **en el volumen data_apache asociado a la ruta /usr/local/apache2/htdocs** creamos dichos directorios en nuestro caso los nombres son primera y segunda.