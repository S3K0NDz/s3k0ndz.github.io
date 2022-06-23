---
layout: post
author: S3konDz
title: Máquina TenTen
image: https://www.hackthebox.com/storage/avatars/e58a465e93364c423cd2162945f8f7bf.png
description: Resolución de máquina
---
<img style="display: block;
  margin-left: auto;
  margin-right: auto;
  width: 50%;" src="https://www.hackthebox.com/storage/avatars/e58a465e93364c423cd2162945f8f7bf.png">
  <br>

La máquina que voy a resolver hoy es Tenten, tiene una dificultad media y toca las siguientes técnicas.

* Enumeración en WordPress
* Job-Manager Wordpress Plugin [CVE-2015-6668]
* Steganografía con la herramienta **Steghide**
* Crakeo hash usando la herramienta **John the Ripper**
* Escalada usando listado de privilegios de sudoers


## Reconocimiento

Para empezar lanzamos una traza a la ip para ver si está operativa la máquina. 


```
ping -c 1 10.10.10.10
```
```
--- 10.10.10.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 38.111/38.111/38.111/0.000 ms
```
En este caso la máquina está encendida por lo que podemos pasar a la fase de enumeración de puertos.

Hice una utilidad para hacer acelerar este proceso, crea una carpeta con el nombre de la máquina y genera 3 subdirectorios (data, explotation y nmap) En este último directorio tendremos el reporte que genere el reconocimiento con nmap, podéis usarla clonando el siguiente repositorio de github. 

```
git clone https://github.com/S3K0NDZ/machine.git
```

Al ejecutar la herramienta os pedirá el nombre de la máquina, la ip y tipo de escaneo que queréis usar, en este caso vamos a usar el completo. 

```
Introduce el nombre de la máquina: Tenten
Introduce la ip de la máquina: 10.10.10.10
Selecciona tipo de escaneo: 
r) Escaneo rápido
c) Escaneo completo
```

Podemos ver como en el reporte que obtenemos, tenemos abiertos los puertos 22 y 80.

```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-21 01:56 CEST
Nmap scan report for 10.10.10.10
Host is up (0.046s latency).
Not shown: 998 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (
Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:f7:9d:38:0c:47:6f:f0:13:0f:b9:3b:d4:d6:e3:11 (RSA)
|   256 cc:fe:2d:e2:7f:ef:4d:41:ae:39:0e:91:ed:7e:9d:e7 (ECDSA)
|_  256 8d:b5:83:18:c0:7c:5d:3d:38:df:4b:e1:a4:82:8a:07 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-generator: WordPress 4.7.3
|_http-title: Job Portal &#8211; Just another WordPress site
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results
at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.14 seconds
```

## Intrusión

Para la intrusión lo primero que vamos a hacer es entrar en nuestro navegador de preferencia y buscar la página que está alojada en el puerto 80. 

```
http://10.10.10.10
```
Y veremos la siguiente web. 

<img src="http://drive.google.com/uc?export=view&id=1lW4aEO2C0yltPR289OQzqHDpzeT_uB71">

Si lo anazalizamos con whatweb podemos ver como la tecnología que usa es WordPress 

```
http://10.10.10.10 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.10], JQuery[1.12.4], MetaGenerator[WordPress 4.7.3], PoweredBy[WordPress,WordPress,], Script[text/javascript], Title[Job Portal &#8211; Just another WordPress site], UncommonHeaders[link], WordPress[4.7.3]
```
Una vez sabemos esto y vemos que tiene una versión desactualizada, podemos listar usuarios y plugins vulnetables con WPscan, yo en este caso voy a listar usuarios con WPSCAN y voy a enumerar plugins con gobuster fuzzeando, usando un diccionario del repositorio de SECLIST.

```
git clone https://github.com/danielmiessler/SecLists
```
De aquí voy a usar el diccionario de **wp-plugins.fuzz.txt** para listar plugins. 

Usando la herramienta WPscan 

```
wpscan --url http://10.10.10.10 -enumerate u
```
He conseguido un usuario que se llama **Takis**.

Ahora con gobuster aplicaré fuzzing con el diccionario nombrado anteriormente para enumerar plugins.

Me llama la atención el plugin job manager, se usa en la siguiente parte de la página para enviar CV. 

foto tenten3 -------------------


, y buscando vulneravilidades por google me he topado con la siguiente. 

foto tenten2 ---------------



## Escalada de privilegios 
## Video
