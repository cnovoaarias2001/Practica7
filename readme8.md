# Configura un sistema cun servidor DNS e un cliente alpine que cumpla os seguintes requisitos.

**1º Volumen por separado da configuración** 

En este caso configuraremos dos volumenes, un chamado ".conf" e ".zonas"<br><br>

**2º Red propia interna para todos los contenedores**  
En esta parte de networks configuraremos unha IP estática para nuestro servidor DNS que usaran nuestros contenedores.<br>

networks:
      bind9_subnet:
        ipv4_address: 172.30.5.1  # IP estática del servidor DNS
<br><br>

**3º Configuración de los Forwarders**  
En este configuraremos en `/etc/bind/named.conf.options` una lista de servidores DNS que resolveran de manera local, google y Cloudflare.<br>

forwarders {
        8.8.8.8;  // Servidor DNS de Google
        1.1.1.1;  // Servidor DNS de Cloudflare
    };<br><br>


**4º Zona propia**  
Este apartado se configura dentro de  `db.carlos.int` dentro do directorio de `zonas` y configuraremos el .yml.  
```
$TTL 38400	; 9 hours 30 minutes
@		IN SOA	ns.carlos.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh 
				3600       ; retry 
				604800     ; expire 
				38400      ; minimum 
				)
@		IN NS	ns.carlos.int.
ns		IN A		172.30.5.1
test	IN A		172.30.5.4
www		IN A		172.30.5.7
alias	IN CNAME	practica7
texto	IN TXT		ejercicio
``` 
<br>
Cliente con IP dentro de la misma subred y con DNS del servidor.
Una vez hecho esto podemos utilizar docker-compose -d up y ejecutaremos dentro del cliente `docker exec -it practica7_alpine /bin/sh`.<br>

`dig 172.30.5.1`<br>

```
; <<>> DiG 9.18.27 <<>> ns.carlos.int
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15843
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8602402d2c684f2f01000000674561a4a0d934342e58c05b (good)
;; QUESTION SECTION:
;ns.carlos.int.			IN	A

;; ANSWER SECTION:
ns.carlos.int.		38400	IN	A	172.30.5.1

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Thu Nov 15 11:11:30 UTC 2024
;; MSG SIZE  rcvd: 84
```