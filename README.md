Movistar IPTV U7D - Flussonic Catchup proxy
===========================================

Qué es y de dónde nace
----------------------

Este proyecto nació del descontento de acceder a los canales de TV de Movistar a través de su [app de Movistar](https://play.google.com/store/apps/details?id=es.plus.yomvi), que dicho de forma elegante, está muy por detrás de la competencia.

 - No tiene ningún tipo de integración real con el Android, ni siquiera para la entrada de datos. Intentad usar un mini teclado, veréis lo _cómodo_ que es. Te dan ganas de llamar al soporte y decirles la categoría de software que venden con su producto.

 - Tiene un click molesto cada vez que pulsas un botón en el mando, no se puede deshabilitar.

 - El vídeo es de menor calidad que el que llega por la VLAN de tv, en lugar de a 50HZ es a 25, y se nota.

 - Se congela todo el tiempo, se cuelga.

 - Debido al magnífico DRM, en el siguiente video, el video nunca se ve: [Movistar+_20210320.mp4](https://openwrt.marcet.info/u7d/Movistar%2B_20210320.mp4)

Durante años usé [udpxy](http://www.udpxy.com/) para acceder a los directos desde cualquier cliente IPTV, aunque estuviera conectado por wifi. Con la lista de canales y programación que podías obtener con el fantástico [tv_grab_es_movistartv](https://github.com/MovistarTV/tv_grab_es_movistartv) no hacía falta nada más que un webserver para servirlo todo. Faltaba el acceso a los últimos 7 días.

Para esto, lo mejor que hubo durante mucho tiempo fue un [addon cerrado](https://sourceforge.net/projects/movistartv/) para el magnífico [Kodi](https://kodi.tv/), que funcionaba relativamente bien. Te permite reproducir programas de la parilla de los últimos 7 días; también te permite hacer grabaciones en local o en la nube, que son accesibles después para reproducir.

Tenía unos cuantos _peros_:
 - Aunque puedes reproducir un programa, no puedes avanzar, ni pausar, ni rebobinar, sólo reproducir y detener.
 - Con frecuencia la reproducción fallaba y aunque a veces podías continuar en el mismo punto, a menudo tenías que volver a repdoducirlo desde el principio.
 - Lo peor de todo eran los microcortes durante la reproducción, imposible de eliminar por completo.

En suma, era usable para grabaciones locales y para directos, para otros usos decepcionaba bastante.

A continuación de ésto, descubrí el [framework de entrada de TV del Android](https://source.android.com/devices/tv/), que es lo que usan todos los grandes fabricantes cuando ofrecen televisores con Android. De no tener SmartTV, hay diferentes TVboxes que hacen *smart* cualquier televisor con HDMI. El Android aporta las bases necesarias para poder acceder a cualquier tipo de contenido. Hay una aplicación simple de ejemplo que viene con el [AOSP](https://source.android.com/), la versión libre del Android que todos los que venden aparatos con Android utilizan. Además, se pueden usar otros clientes (que a su vez utilizan dicho framework). Por lo que sé, Sony y Phillips tienen sus propios clientes, aunque no los he probado.

Un poco más adelante, descubrí el que creo que es, si no el mejor, uno de los mejores clientes para acceder a la TV. El [TiviMate](https://play.google.com/store/apps/details?id=ar.tvplayer.tv).

Pasó a ser mi modo favorito de acceder a los canales, nada se le acercaba. Es super fluido, te permite hacer no solo PiP de lo que estás viendo, sino que puedes estar visionando hasta 9 canales simultáneos como si fuera un sistema de cámaras de seguridad.

Tan contento con él estaba que tenía que poder usar el resto de funcionalidad. Daba acceso a servicios de catchup (últimos 7 días), y lo hacía de dos maneras diferentes. Después de hacer pruebas, monitorear qué conexiones realizaba cuando intentabas ver algo de los últimos 7 días, ...

Se me ocurrió que podía hacer algún tipo de proxy entre dicho TiviMate con catchup flussonic y la IPTV de Movistar (de las variantes de catchup que soporta es el que más extendido he encontrado y a la vez es el más sencillo e intuitivo de implementar).

El resultado es algo así (el funcionamiento real es fluido todo el tiempo, el video se llega a atascar en los momentos de mayor tráfico de datos):

 - [TiviMate_Movistar_20210320_U7D-1.mp4](https://openwrt.marcet.info/u7d/TiviMate_Movistar_20210320_U7D-1.mp4)

 - [TiviMate_Movistar_20210320_U7D-2.mp4](https://openwrt.marcet.info/u7d/TiviMate_Movistar_20210320_U7D-2.mp4)


Observaciones
-------------

 - Sólo hace falta tener contratada la fibra de Movistar, no es necesario ningún paquete de televisión.

 - La funcionalidad que más trabajo me dio conseguir y que más agradezco a la hora de usarlo es la reproducción continua. ¿Que qué es eso? Pues dado que Movistar da acceso a la programación de los últimos 7 días a partir de un identificador de canal y un identificador de programa, ambos incluidos en la EPG, a la hora de reproducir cualquier momento de la última semana se establece una negociación que te da acceso a reproducir **ese** programa, no el siguiente. De esta manera, tanto en la app oficial como en el addon cerrado, reproduces un programa y al acabar (normalmente sobre 1 o 2 minutos después del final) se detiene.

Con este servicio, en lugar de cortarse, se produce una mínima pausa, durante la que se detiene el sonido y la imagen queda congelada. Es durante un mínimo instante de tiempo, un par de segundos, por lo que aunque te das cuenta de que ha habido un cambio de programa, no molesta lo más mínimo. Así puedes ver toda la programación de un canal en diferido, el tiempo que quieras dentro de la última semana, sin más que iniciar la reproducción en el instante deseado.


Componentes
-----------

El resultado son dos microservicios escritos en python asíncrono, con [Sanic](https://github.com/sanic-org/sanic):

 - `movistar-u7d.py`: el proxy principal con el que se comunica el cliente final, como el TiviMate.

 - `movistar-epg.py`: otro miscroservicio en python asíncrono encargado de localizar el programa correspondiente al punto temporal que solicita el cliente. Mantiene el estado necesario para el servicio, permitiendo que el microservicio principal no tenga estado y pueda trabajar en múltiples hilos sin problemas.

 - `u7d.py`: pequeño script que mantiene abierta la reproducción de los programas de los últimos 7 días, habrá uno en ejecución por cada programa que se esté visionando, de consumo inapreciable.

 - `tv_grab_es_movistartv`: encargado de generar la lista de canales y la programación, así como de guardar una caché de los últimos 8 días de programas, de manera que necesita ser ejecutado de forma recurrente (cada 2h). Esta información es imprescindible para que todo el proceso funcione bien. Tanto TiviMate como cualquier repdoductor con catchup flussonic sólo se preocupan por canal y un timestamp, que define un momento preciso en el tiempo. El proxy es el encargado de encontrar qué programa de la EPG corresponde a ese canal en ese momento y negociar con Movistar la reproducción.

 - `env-example`: fichero fundamental. Contiene las variables de entorno con el que configurar las diferentes partes. Para usarlo con docker, basta con copiarlo a `.env` y hacer los cambios necesarios. La primera parte de estas variables sólo hacen falta para usarlo dentro del container, la segunda mitad en cambio contiene variables necesarias para ejecutarlo de un modo u otro.

El resto de ficheros:

 - `updateguide.sh`: script de ejemplo para ejecutar desde el host y de forma recurrente, si el servicio se ejecuta dentro del docker
 - `crontab`: crontab de ejemplo para ejecutar el script anterior, cada 2h, en el minuto 5. La frecuencia es así de alta para enterarse de los cambios de última hora que a veces sufre la programación.


Instalación
-----------

Tenemos la opción de utilizar docker y docker-compose. Dentro del container queda casi todo lo necesario. En el host necesitamos:

 - [udpxy](http://www.udpxy.com/), que debe estar configurado para que pueda acceder a los canales de Movistar;

```
udpxy -T -S -a br-lan -p 4022 -m eth0.2 -c 20 -B 7896
```

 - Un `crontab`, es decir, algo que periódicamente llame al script `updateguide.sh` si usamos Docker, o `tv_grab_es_movistartv` directamente.

 - [igmpproxy](https://github.com/pali/igmpproxy) para que `tv_grab_es_movistartv` funcione correctamente dentro del container.

```
$ cat /etc/igmpproxy.conf
quickleave

phyint eth0.2 upstream ratelimit 0 threshold 1
        altnet 172.0/11

phyint br-tvlan downstream ratelimit 0 threshold 1
```

Donde `eth0.2` es la VLAN 2 de Movistar, la de IPTV y `br-tvlan` es la subred `tvlan` en el docker-compose.

Para que el microservicio `movistar-u7d.py` sea accesible desde la lan, tendremos que modificar el firewall de manera que los paquetes que lleguen al puerto 8888 del host sean redirigidos al puerto 8888 de la ip donde escucha `movistar-u7d.py`, en todos los ejemplos U7D_ADDRESS. En OpenWrt bastaría con añadir a `/etc/config/firewall`:

```
config redirect
        option name 'DNAT_LAN_TO_DOCKER_MOVISTAR_U7D'
        option src 'lan'
        option src_dport '8888'
        option dest 'tvlan'
        option dest_ip '10.17.0.3'
        option dest_port '8888'
        option proto 'tcp'
        option target 'DNAT'
```

que corresponde a:

```
iptables -A zone_lan_prerouting -p tcp -m tcp --dport 8888 -m comment --comment "!fw3: DNAT_LAN_TO_DOCKER_MOVISTAR_U7D" -j DNAT --to-destination 10.17.0.3:8888
```

Por desgracia, al tener dos subredes dentro del container, no he conseguido hacerlo funcionar con el mapeado de puertos del propio docker, que haría infinitamente más sencillo este paso.



En el caso de querer ejecutarlo todo directamente, pues:

```
pip3 install -r requirements.txt
```

Y a partir de ahi lanzarlo con algo como `start.sh`, que está pensado para el docker, pero sirve de ejemplo. Sin olvidarnos del crontab anterior, la EPG siempre la necesitamos actualizada.

Cualquier duda o consulta no dudéis en abrir una [incidencia](https://github.com/jmarcet/movistar-u7d/issues) [aquí](https://github.com/jmarcet/movistar-u7d) en Github.


Uso
---

Sólo queda configurar el cliente. Para eso tenemos las siguientes URLs, donde 192.168.1.1 es la ip local del host donde tengamos el proxy funcionando.

 - Canales: `http://192.168.1.1:8888/channels.m3u` o `http://192.168.1.1:8888/MovistarTV.m3u`

 - Guía de programación (EPG): `http://192.168.1.1:8888/guide.xml`

Con configurar esas dos cosas debería ser suficiente. Aseguráos de que el TiviMate (o cliente IPTV con catchup Flussonic) guarda al menos 8 días de historial del EPG.


Posibles problemas
------------------

A veces se desincroniza la guía entre el cliente (TiviMate) y el proxy, mostrando `Error 404` en todo o en casi todo. La solución pasa por ir a los ajustes del cliente (TiviMate), borrar la EPG y cargarla de nuevo.

Del mismo modo, a veces hay un mínimo glitch en la guía, no causado en sí por el proxy sino por lo inestable de los metadatos que Movistar envía, a veces incompletos y con cambios frecuentes de última hora. El efecto es que en alguna ocasión pueden quedar huecos sin programación, incluso un día entero. No es habitual, pero tengo que ver aún cómo arreglarlo del todo.


Agradecimientos
---------------

Sin ningún orden en especial:

- [_WiLloW_](https://github.com/MovistarTV): por su [tv_grab_es_movistartv](https://github.com/MovistarTV/tv_grab_es_movistartv), un trabajo increíble que desenmaraña todos los metadatos de MovistarTV.

- [XXLuigiMario](https://github.com/XXLuigiMario): por su [u7d.py](https://github.com/XXLuigiMario/MovistarU7D) que fue, en cierta manera, el punto de partida de todo, aunque le faltase media funcionalidad. Partiendo de él y analizando el tráfico del [addon cerrado](https://sourceforge.net/projects/movistartv/), con todos sus fallos, conseguí entender e implemnetar correctamente la negociación rtsp de la que depende toda reproducción de los últimos 7 días.

- [Sanic Framework](https://sanicframework.org/): hace comodísima la programación de microservicios web.


Futuro
------

Tenía pensado reescribirlo todo en Golang. Movistar, sin embargo, sigue añadiendo DRM a los canales, no contento con los propios de su plataforma, continúa con la TDT. En mi comunidad autónoma ya no se ven los regionales, así que le quedará poco tiempo de uso a todo esto lamentablemente.

[Aquí](https://comunidad.movistar.es/t5/Soporte-M-D-Yomvi/Por-favor-no-encripteis-los-canales-de-TDT/m-p/4437418#M107537) podéis ver un intento vano de hacerlos entrar en razón.
