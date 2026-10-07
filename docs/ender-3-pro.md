# μprint con una Creality Ender 3 Pro

Guía para conectar μprint a una Ender 3 Pro (placa 4.2.x) y mandar impresiones por USB sin tocar el firmware de la impresora.

## Cómo funciona

El ESP32-S3 se conecta al puerto micro-USB de la Ender, el que está al lado de la ranura de la SD. La placa 4.2.x de la Ender tiene un conversor USB-serial CH340 que va directo al UART del STM32. μprint actúa como USB host: ya trae el driver CH34x y por defecto usa 115200 baudios, la misma velocidad que el Marlin de Creality.

Subís el G-code desde el navegador o el slicer, μprint lo guarda en la microSD del ESP (o en la memoria interna, en la variante N16R8) y lo manda a la impresora línea por línea, con número de línea y checksum, esperando el "ok" de cada una antes de mandar la siguiente. No se toca la placa de la impresora ni su SD, y no hace falta modificar el firmware de la Ender.

## Qué comprar

| Ítem | Detalle |
| --- | --- |
| ESP32-S3 DevKit | Con puerto USB nativo (OTG). La mayoría trae dos USB-C, marcados "UART/COM" y "USB/OTG". Recomendado: N16R8 (16 MB flash, 8 MB PSRAM), que suma unos 12 MB de memoria interna. Aun así conviene la microSD, porque un G-code grande de la Ender puede superar eso. |
| Cable/adaptador a la impresora | Un adaptador OTG USB-C macho → USB-A hembra más un cable USB-A → micro-USB común (que pase datos, no uno solo de carga), o un cable OTG USB-C → micro-USB directo. |
| Fuente de alimentación | 5 V, cargador de celular de 1 A o más, con un cable USB-C para el puerto UART/COM. |
| Módulo microSD | SPI, con tarjeta formateada en FAT32, más cables dupont hembra-hembra (opcional en la variante N16R8). |
| Cinta Kapton | O cinta aislante fina. |

Aclaración: el ESP32 clásico, el C3 y el C6 NO sirven, porque no tienen USB host.

## Cableado de la SD

Pines por defecto (se pueden cambiar en Ajustes):

| Señal | Pin del ESP32-S3 |
| --- | --- |
| MOSI | GPIO11 |
| MISO | GPIO13 |
| SCLK | GPIO12 |
| CS | GPIO10 |
| GND | GND |
| VCC | 5V si el módulo tiene regulador (la mayoría de los baratos traen un AMS1117); 3V3 si no. Revisá tu módulo. |

## Puertos

El puerto UART/COM se usa para alimentar el ESP32 y para flashearlo. El puerto USB/OTG va conectado a la impresora.

## Alimentación

Esto difiere de lo que dice el README original del proyecto (en alemán), que pide cerrar el puente de soldadura "USB-OTG" del S3 para que el puerto OTG entregue 5 V. Para la Ender 3 Pro recomendamos lo contrario:

- NO cierres el puente "USB-OTG" del S3.
- Tapá con Kapton el pin VBUS del conector USB-A del cable que va a la impresora.

Así por ese cable solo pasan datos y GND. Las placas Creality toman 5 V del USB (por eso el LCD se prende con solo enchufar el USB), así que si el S3 entregara 5 V por ese cable, alimentaría a medias la placa de la Ender. Con la cinta, el CH340 se alimenta desde la propia placa y μprint detecta la impresora cuando la prendés. Es el mismo truco que se usa con OctoPrint y una Raspberry Pi.

Para identificar VBUS sin depender de diagramas: con el cable enchufado SOLO a la impresora y la impresora prendida, medí con un tester en DC entre los dos pines de los extremos del conector USB-A (son los más largos). Si marca +5 V, la punta roja está en VBUS: ese es el pin que tapás con Kapton. Si marca −5 V, VBUS es el de la punta negra. Los dos pines del medio son datos y no se tocan.

## Flashear

1. Abrí [notluken.github.io/uprint](https://notluken.github.io/uprint/) en Chrome o Edge en una compu (Firefox y Safari no soportan WebSerial).
2. Conectá el ESP32-S3 al puerto UART/COM con un cable de datos.
3. Elegí la variante Basic o N16R8, dale a Instalar y elegí el puerto serie en el diálogo del navegador; la primera vez confirmá que borre el dispositivo.
4. Al terminar, apretá RESET.

Si no lo detecta: mantené BOOT presionado, tocá RESET y soltá BOOT; después probá instalar de nuevo.

Alternativa: el flasher original en [dmyrenne.github.io/uprint](https://dmyrenne.github.io/uprint/) funciona igual, pero sin la UI en español.

## Primera configuración

Conectate a la red Wi-Fi "uprint" (contraseña `uprint123`) y abrí http://192.168.4.1. En la pestaña Wi-Fi elegí tu red de casa. Después vas a encontrar μprint en http://uprint.local.

En Android, `.local` a veces no resuelve; en ese caso usá la IP que muestra la página o la que aparece en el router.

Recomendado: cambiar la contraseña del access point en Ajustes.

## Ajustes para la Ender 3 Pro

Los valores por defecto ya le sirven a la Ender 3 Pro, no hace falta tocar nada en Marlin:

- Velocidad (baudios): 115200.
- Elevación de boquilla: 5 mm al pausar, 10 mm al cancelar.
- Posición de estacionamiento: X0 Y200, que deja la cama hacia adelante (la Ender 3 Pro tiene 220×220 mm de área de impresión).

## Imprimir

Subí el archivo en la pestaña Subir, elegilo en Archivos y apretá Imprimir. Pausar, Reanudar y Cancelar están en la pestaña Impresión.

Limitaciones:

- El archivo NO aparece en el menú del LCD de la impresora.
- La impresión depende del ESP: si se reinicia o se queda sin alimentación, la impresión se corta.
- Mientras μprint está imprimiendo, no arranques otra impresión desde el LCD.

## Desde el slicer

- PrusaSlicer: agregá una impresora física, tipo de host PrusaLink, hostname `uprint.local`, clave de API desde Ajustes → Slicer.
- OrcaSlicer: tipo de host OctoPrint, con los mismos datos.
- Cura: plugin "OctoPrint Connection" del Marketplace. No probado con μprint.

`.bgcode` (G-code binario) no está soportado: si tu perfil lo usa, desactivalo en la configuración de impresora del slicer.

## Actualizar

Bajá el archivo `uprint-…-ota.bin` de la variante correcta (`basic` o `n16r8`) desde [github.com/notluken/uprint/releases](https://github.com/notluken/uprint/releases) y cargalo en Ajustes → Firmware. Se conservan los ajustes, el Wi-Fi y los archivos.

## Problemas comunes

**No detecta la impresora.** Revisá que esté prendida, que el cable esté en el puerto OTG del ESP (no en el UART) y que el cable o adaptador pase datos y sea OTG. Último recurso: cerrar el puente "USB-OTG" y sacar la cinta Kapton. Con esto el ESP va a alimentar en parte la placa de la Ender cuando esté apagada.

**Mensajes de error en alemán.** Algunos mensajes de error de μprint (archivo inexistente, memoria llena, SSID inválido, etc.) todavía salen en alemán. Los genera el firmware del ESP y no la interfaz web, así que no entran en la traducción. Es una limitación conocida.

**Micro-pausas en curvas con mucho detalle.** μprint manda una línea de G-code por vez, así que en curvas muy detalladas pueden aparecer micro-pausas. Si pasa, probá bajar la resolución en el slicer ("Maximum resolution" en Cura, "Resolution" en PrusaSlicer/Orca).

## Créditos

μprint es de Daniel Myrenne (licencia MIT). Este fork agrega la interfaz en español y esta guía.
