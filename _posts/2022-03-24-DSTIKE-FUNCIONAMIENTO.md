---
layout: post
author: S3konDz
title: Uso y review del reloj DSTIKE DEAUTHER mini
description: Prueba descripción
---
Considero que no hay una forma mas interesante de estrenar este rincóncito que tengo en internet, que compartir el funcionamiento de uno de los cacharritos
que mas me han llamado la ateción desde que me metí en el mundo del hacking, estoy hablando del reloj DSTIKE, un dispositivo portatil con forma de reloj que te 
permite hacer varios tipos de ataques jugando con la redes wifi de 2.4 GHz. 

Este es el dispositivo en cuestión: 

<img src="https://ae01.alicdn.com/kf/H79491ed834ec4bed901e4931f7a6bfdbQ.jpg">

El reloj funciona sobre un micro **ESP8266** con módulo wifi 2.4 GHz incorporado.

Para empezar voy a enumerar todas las funcionalidades que tiene este reloj, y así saber exáctamente lo que podemos hacer y lo que no con este dispositivo.

### Reconocimiento de redes 

El primer modo que tiene el DSTIKE es el modo **SCAN**, este modo nos permite como su propio nombre indica escanear las diferentes redes y así obterner sus SSID, 
además de esto nos deja escanear por estaciones conectados a las diferentes APs para así obtener sus direcciones MAC. 


<img src="http://drive.google.com/uc?export=view&id=14TWYjX1aQS1nNS8sGw9VC-U6oAqaHcQP">


Como se puede ver en la siguiente imagen tenemos 3 tipos de escaneos distintos, mi recomendación es que siempre usemos el primero y de esta forma matamos 2
pájaros de un tiro ya que así podremos escanear las diferentes APs y sus estaciones.


<img src="http://drive.google.com/uc?export=view&id=10IJ2eT7eYzYrsutM8zb-jRvtgiwRDcBI">

<img src="http://drive.google.com/uc?export=view&id=1-nmqjWtu0phccrTzI8ifpMreB-OjDeSR">


### Select

El segundo modo que tiene el DSTIKE es el modo **SELECT**, este modo nos permite seleccionar de manera individual las APs y las estaciones que queremos atacar.
Además de esto tiene un modo llamado SSID que nos permite randomizar hasta 60 SSID para un ataque **Beacon**, del cual os hablaré mas adelante. 

<img src="http://drive.google.com/uc?export=view&id=1QbYqTJ0FHR2vjzZCG9Lhe620hdlbXX-H">

En este caso estoy seleccionando mi red personal MOVISTAR xxxx


<img src="http://drive.google.com/uc?export=view&id=1ic5j9qGRDPXVTqanDGR_3PhkbqxgIejR">


### Diferentes ataques 

El tercer modo que tiene el DSTIKE es el modo **Atack**, este modo nos permite hacer ataques como los siguientes: 

<img src="http://drive.google.com/uc?export=view&id=12CyUSiy4H7CEHr3SeYljn5388mDZpYzS">

<img src="http://drive.google.com/uc?export=view&id=1QJgGx-3SAWRLODMBqIy1XlcGxAnDZKre">

#### Deauth

Este ataque nos permite mandar paquetes de desautenticación a las APs o Estaciones que hayamos seleccionado previamente, haciendo que mientras este ataque esté funcionado sea imposible por parte de la víctima volverse a conectar al Wi-fi.

#### Beacon

Este ataque se usa para saturar la red con SSIDs que no conectan a ningún lado, en este caso el reloj permite generar hasta 60 redes para saturar todo el espectro.

#### Probe

Probe Spam se refiere a todas las solicitudes de sondeo que se realizan, esto hace referencia a cuando un dispositivo envía peticiones en busca de la red a la que se quiere conectar todo esto por medio del SSID y de la MAC que se tiene. Básicamente consiste en enviar peticiones de conexión.


### Packet monitor

Este modo es un monitor de tráfico de paquetes de los 14 canales.

<img src="http://drive.google.com/uc?export=view&id=1wox9lHmak5aaJFvutEo_PK9O20rWEciI">

### Reloj

Como todo reloj que se precie tiene que dar la hora, este no podía ser menos, por lo que podemos ver y ajustar la hora cuando queramos.

<img src="http://drive.google.com/uc?export=view&id=1GNF-NSvdXs4wVqNsR9UhKjiWtI-zwW0Z">

### Led

Por último el modo mas útil de todos, una luz led por si nos quedamos sin iluminación y las cosas se ponen chungas. 

<img src="http://drive.google.com/uc?export=view&id=15ZhBReXFhiuBElQqMvj9E4Hmacv7Q5aP">

Un saludo a los lectores! 
