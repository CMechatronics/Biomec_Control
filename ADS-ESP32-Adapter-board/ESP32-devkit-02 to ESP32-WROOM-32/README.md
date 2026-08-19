# Placa adaptadora: ESP32-DevKit-C3-02 → ESP32-WROOM-32

## Resumen ejecutivo

Este directorio contiene el diseño y documentación de la placa adaptadora que permite migrar del ESP32-C3 DevKitC-02 al ESP32-WROOM-32, manteniendo compatibilidad total con la interfaz del ADS1198 y reduciendo la dependencia de hardware específico.

## Motivación técnica

El ESP32-C3 DevKitC-02 presenta limitaciones de disponibilidad en el mercado y restricciones en número de pines GPIO. El ESP32-WROOM-32 ofrece:
- Mayor capacidad de procesamiento (2 núcleos vs 1)
- Más pines GPIO disponibles (34 vs 13)
- Mayor memoria RAM y flash
- Mejor disponibilidad en el mercado
- Mejor rendimiento energético en aplicaciones con WiFi/Bluetooth

La placa adaptadora permite cambiar de microcontrolador sin modificar la base del sistema ni el firmware de procesamiento de señal.

---

## Especificaciones técnicas comparativas

### Arquitectura y procesamiento

| Aspecto | ESP32-C3 DevKit-02 | ESP32-WROOM-32 |
|--------|-------------------|------------------|
| **Arquitectura** | RISC-V 32-bit | Xtensa 32-bit dual-core |
| **Núcleos** | 1 @ 160 MHz | 2 @ 80/160 MHz (configurable) |
| **Caché** | 16 KB I + 8 KB D | 32 KB I + 32 KB D (per core) |
| **Memoria RAM** | 400 KB | 520 KB |
| **Flash externo** | 4 MB | 4/8/16 MB (configurable) |
| **Voltaje** | 3.3 V | 3.3 V |
| **Consumo activo** | ~40-60 mA | ~80-100 mA (dual-core) |

### Conectividad

| Característica | ESP32-C3 | ESP32-WROOM-32 |
|---|---|---|
| **WiFi** | 802.11 b/g/n (2.4 GHz) | 802.11 b/g/n (2.4 GHz) |
| **Bluetooth** | LE 5.0 only | Classic 5.0 + LE 4.2 |
| **Antena** | Integrada PCB | Integrada PCB |
| **TX Power** | +15 dBm (WiFi), +4 dBm (BLE) | +20 dBm (WiFi), +4 dBm (BLE) |

### Periféricos disponibles

| Periférico | ESP32-C3 | ESP32-WROOM-32 |
|---|---|---|
| **UART** | 2 (GPIO20/21, GPIO5/6 alt) | 3 (GPIO1/3, GPIO9/10, GPIO17/16 alt) |
| **SPI** | 1 maestro (GPIO6/7/2/10) | 3 (SPI2 en GPIO12-15, SPI3 en GPIO17-22) |
| **I²C** | 2 | 2 (GPIO21/22) |
| **GPIO analógicos** | 11 canales ADC | 12 canales ADC (algunos compartidos con SPI) |
| **PWM** | 6 canales | 16 canales |
| **Timer** | 2 | 4 |
| **RTC** | Sí | Sí |

---

## Mapa de equivalencias GPIO

### Interfaz del ADS1198 (SPI principal)

| Función | ESP32-C3 DevKit-02 | ESP32-WROOM-32 | Notas |
|---------|-------------------|-----------------|-------|
| **FE_RESET** | GPIO0 | GPIO32 | Reinicio del front-end ADS1198 |
| **FE_DRDY** | GPIO1 | GPIO35 | Data Ready del ADS1198 |
| **SPI_MISO** | GPIO2 | GPIO12 | ESP32 ← ADS1198 (datos) |
| **FE_START** | GPIO3 | GPIO33 | Inicio de conversión del ADS |
| **SPI_CLK** | GPIO6 | GPIO14 | Reloj SPI |
| **SPI_MOSI** | GPIO7 | GPIO13 | ESP32 → ADS1198 (datos) |
| **FE_CS** | GPIO10 | GPIO15 | Chip Select del ADS1198 |

### Pines de soporte general

| Función | ESP32-C3 | ESP32-WROOM-32 | Observaciones |
|---------|----------|---|---|
| **3V3** | Pin 2 | Pin 2 | Alimentación lógica |
| **5V** | Pin 1 | Pin 1 | Entrada de alimentación externa |
| **GND** | Pins 3, 4, 5 | Pins 3, 4, 5 | Referencia común (múltiples puntos) |
| **USB D+** | GPIO19 | GPIO25 | No usado en este diseño |
| **USB D-** | GPIO18 | GPIO26 | No usado en este diseño |
| **EN/RST** | GPIO9 | GPIO EN | Reinicio del microcontrolador |
| **BOOT** | GPIO8 | GPIO0 alt | Modo de arranque |

### GPIO disponibles adicionales

Pines libres para futuras expansiones o depuración:

**ESP32-WROOM-32:**
- GPIO4, GPIO5, GPIO23, GPIO27, GPIO34, GPIO36, GPIO37, GPIO38, GPIO39
- GPIO25, GPIO26 (para USB si se requiere, no usado actualmente)

---

## Consideraciones de diseño

### Alimentación

- **Voltaje de entrada recomendado:** 5V ± 0.5V
- **Regulador:** LDO de bajo ruido para generar 3.3V
- **Capacitores:** 
  - Entrada: 10 µF (para estabilidad de línea de 5V)
  - Salida: 10 µF + 100 nF (cerca del ESP32-WROOM-32)
- **Consumo pico esperado:** ~150 mA (con WiFi activo)

### Aislamiento de ruido

El ADS1198 es un front-end de bajo ruido sensible a perturbaciones electromagnéticas:

- Separación física entre zona de alimentación del ADS1198 y ESP32-WROOM-32
- Plano de GND continuo bajo el ADS1198
- Vías de paso (via stitching) alrededor del área de señal analógica
- Capacitores de desacoplamiento locales (100 nF) para cada pin de alimentación del ADS

### Interfaces SPI

**ESP32-WROOM-32 SPI2 (usado):**
- Frecuencia configurada: 2 MHz (conservadora para estabilidad)
- Bus compartido con la interfaz SD (cuidado si se agrega soporte SD)
- Modo: SPI Master, CPOL=0, CPHA=0

**Líneas de control especiales:**
- **DRDY** es una línea de interrupción activa baja
- **CS** debe mantenerse baja durante toda la transacción SPI
- **RESET** debe pulsarse al menos 2 µs

### Protección ESD

- Protección ESD serie en líneas de señal analógica (optional, depende de ambiente)
- Protección TVS en líneas de GPIO si están expuestas

---

## Estructura del proyecto

```
ESP32-devkit-02 to ESP32-WROOM-32/
├── docs/
│   ├── planteamiento.pdf          # Análisis y justificación técnica
│   └── README.md                  # Este archivo
├── hardware/
│   └── kicad/
│       ├── esp32c3-esp32Wroom/   # Diseño principal de la placa
│       │   ├── esp32c3-esp32Wroom.kicad_sch
│       │   ├── esp32c3-esp32Wroom.kicad_pcb
│       │   ├── esp32c3-esp32Wroom.kicad_pro
│       │   └── .history/          # Historial de cambios
│       └── MiESP32Lib/            # Librería de símbolos personalizados
│           └── MiESP32Lib.kicad_sym
├── src/                           # Firmware en desarrollo
├── lib/                           # Librerías reutilizables
├── signal_processing/             # Módulos de procesamiento de señal
└── tools/                         # Scripts y utilidades
```

---

## Cambios de firmware necesarios

### Configuración de pines en PlatformIO/Arduino

```cpp
// Configuración SPI
#define SPI_MISO  12
#define SPI_MOSI  13
#define SPI_CLK   14
#define SPI_CS    15

// Señales de control del ADS1198
#define ADS_RESET 32
#define ADS_DRDY  35
#define ADS_START 33

// Verificar que el SPI esté configurado para SPI2:
// hspi_host en lugar de vspi_host
```

### Inicialización del SPI

```cpp
spi_bus_config_t buscfg = {
    .mosi_io_num = SPI_MOSI,
    .miso_io_num = SPI_MISO,
    .sclk_io_num = SPI_CLK,
    .quadwp_io_num = -1,
    .quadhd_io_num = -1,
    .max_transfer_sz = 4094,
};

esp_err_t ret = spi_bus_initialize(HSPI_HOST, &buscfg, DMA_CHAN_AUTO);
```

---

## Validación y pruebas

### Checklist de verificación hardware

- [ ] Continuidad en pistas de alimentación (5V, 3.3V, GND)
- [ ] Ausencia de cortocircuitos entre pistas de señal
- [ ] Voltaje de salida del regulador: 3.3V ± 0.1V
- [ ] Continuidad en líneas SPI (CLK, MOSI, MISO, CS)
- [ ] Continuidad en líneas de control (RESET, DRDY, START)
- [ ] Resistencia de Pull-up en DRDY (si está configurado)

### Pruebas funcionales

1. **Lectura de registro de identity del ADS1198**
   - Leer registro 0x00 (debe retornar ID del chip)
   - Repetir 10 veces para validar comunicación estable

2. **Adquisición de datos**
   - Configurar ADS1198 en modo continuo
   - Leer datos continuamente
   - Verificar rango esperado (±12V entrada → ±32768 digital)

3. **Rendimiento**
   - Medir frecuencia de muestreo efectiva (debe ser ≥ 1000 Hz)
   - Medir latencia de DRDY a lectura completa

4. **WiFi/Bluetooth** (si se requiere)
   - Verificar conectividad WiFi
   - Verificar rango de señal
   - Evaluar impacto en ruido de ADC

---

## Diferencias operacionales clave

### Ventajas del ESP32-WROOM-32

✅ **Dual-core:** Posibilidad de ejecutar procesamiento pesado en core 1 mientras core 0 gestiona SPI  
✅ **Más memoria:** Espacio para algoritmos de filtrado más complejos  
✅ **Mejor rendimiento WiFi:** Core dedicado disponible  
✅ **Mayor disponibilidad:** Componente más común en el mercado  
✅ **Menores costos:** Precio más estable y comprobado  

### Limitaciones a considerar

⚠️ **Consumo superior:** ~50 mA adicionales con dual-core activo  
⚠️ **Mayor tamaño:** PCB del módulo es más grande  
⚠️ **Configuración SPI:** Usa SPI2 en lugar de SPI1  
⚠️ **Pin layout diferente:** Requiere rediseño de placa adaptadora  
⚠️ **ADC compartido:** Algunos pines ADC comparten funciones con SPI (GPIO12, GPIO13, GPIO14, GPIO15)

---

## Hoja de ruta

| Fase | Tarea | Estado | Target |
|------|-------|--------|--------|
| 1 | Diseño esquemático y layout PCB | ✅ Completado | - |
| 2 | Fabricación de prototipo | ⏳ En progreso | 2-3 semanas |
| 3 | Validación eléctrica del prototipo | ⏳ En progreso | - |
| 4 | Adaptación de firmware a pines nuevos | ⏳ En progreso | - |
| 5 | Pruebas funcionales del sistema completo | ⏹️ Pendiente | - |
| 6 | Generación de PCB final | ⏹️ Pendiente | - |
| 7 | Documentación de fabricación | ⏹️ Pendiente | - |

---

## Referencias y recursos

### Datasheets
- [ESP32-C3 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf)
- [ESP32 (WROOM-32) Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
- [ADS1198 Datasheet](https://www.ti.com/lit/ds/symlink/ads1198.pdf)

### Librerías y soporte
- [ESP-IDF (Espressif IoT Development Framework)](https://github.com/espressif/esp-idf)
- [Arduino-ESP32](https://github.com/espressif/arduino-esp32)
- [KiCad Documentation](https://docs.kicad.org/)

### Herramientas recomendadas
- KiCad 6.0+ para editar esquemáticos y PCB
- PlatformIO o Arduino IDE para programación
- Logic Analyzer de bajo coste para debug SPI

---

## Contacto y soporte

Para consultas técnicas sobre este diseño, consultar la documentación de Biomec UMA o la rama `develop` del repositorio.

**Última actualización:** Agosto 2026  
**Autor:** Departamento de Control, Biomec UMA  
**Licencia:** Ver archivo LICENSE en la raíz del repositorio
