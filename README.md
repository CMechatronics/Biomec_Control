# Biomec Control

Placa adaptadora intermedia entre la PCB del ADS1198 y un ESP32 compatible.

## Resumen

Este proyecto busca mejorar la compatibilidad entre distintas placas ESP32 que se comunican con el ADS1198. La idea es disponer de una interfaz intermedia que permita cambiar de microcontrolador sin rediseñar todo el sistema cada vez que un modelo deje de estar disponible o presente problemas de suministro.

## Objetivo

Aumentar la flexibilidad y la mantenibilidad del sistema, reduciendo la dependencia de una sola placa ESP32. Esto facilita la sustitución de hardware en caso de avería y mejora la disponibilidad de recambios en el taller.

## Contexto técnico

El ADS1198 es un front-end analógico de bajo ruido, con 8 canales y 16 bits, orientado a mediciones ECG/EEG. Su control se realiza desde el ESP32 mediante SPI y algunas líneas GPIO. Además, el sistema contempla una separación entre la zona no aislada y la zona aislada de la placa.

## Base de partida

La primera rama del proyecto parte de un ESP32-C3 DevKitC-02 como referencia de hardware. A partir de ese punto se toma como base la siguiente configuración:

- Microcontrolador ESP32-C3 de arquitectura RISC-V de un solo núcleo.
- Conectividad Wi-Fi 2.4 GHz y Bluetooth LE 5.
- Alimentación por 3.3 V y 5 V en la placa de desarrollo.
- Puerto USB para programación y depuración.
- Pines GPIO expuestos para SPI, control y señales auxiliares.

### Mapa de señales usado en el proyecto

| Señal del proyecto | GPIO | Función |
| --- | --- | --- |
| FE_RESET | GPIO0 | Reinicio / control del ADS |
| FE_DRDY | GPIO1 | Señal de datos listos |
| SPI1_MISO | GPIO2 | Entrada de datos SPI |
| FE_START | GPIO3 | Inicio de conversión |
| SPI1_SCLK | GPIO6 | Reloj SPI |
| SPI1_MOSI | GPIO7 | Salida de datos SPI |
| FE_CS | GPIO10 | Chip select del ADS |

### Pines de soporte de la placa

- 3V3 para alimentación lógica.
- 5V para entrada de alimentación de la placa de desarrollo.
- GND como referencia común.
- USB D+ y USB D- para comunicación serie / JTAG según la configuración.
- EN / RST y BOOT para arranque y programación.

## Motivación

El uso de una placa adaptadora permite trabajar con diferentes modelos de ESP32 sin modificar la base del sistema. Esto reduce la fragilidad ante discontinuidades de fabricación o cambios de disponibilidad, como puede ocurrir con determinadas versiones del ESP32, por ejemplo la DevKit-02 o variantes concretas del C3.

## Líneas de trabajo previstas

- Validar el funcionamiento con distintos modelos de ESP32.
- Probar el rendimiento del ESP32 C3 SuperMini sobre la placa actual.
- Preparar el diseño en KiCad para futuras revisiones y prototipos.

## Herramientas

- KiCad para el diseño electrónico.
- Prototipado previo antes de fabricar la versión final.

## Estado del proyecto

El proyecto se encuentra en fase de definición y validación. El siguiente paso es comprobar la compatibilidad real con distintos ESP32 antes de cerrar una versión definitiva de la PCB.