# Comunicación SPI y análisis del código con MAX6675

## ¿Qué es SPI?

**SPI (Serial Peripheral Interface)** es un protocolo de comunicación síncrono desarrollado por Motorola. Se utiliza para transferir datos entre un microcontrolador (como el STM32) y periféricos como sensores, pantallas, memorias, etc. SPI funciona con una arquitectura maestro-esclavo y utiliza las siguientes líneas:

- **MOSI (Master Out Slave In)**: línea por donde el maestro envía datos al esclavo.
- **MISO (Master In Slave Out)**: línea por donde el esclavo envía datos al maestro.
- **SCLK (Serial Clock)**: señal de reloj generada por el maestro.
- **SS o CS (Slave Select o Chip Select)**: línea para seleccionar el esclavo activo.

### Características de SPI
- Comunicación **full-dúplex** (envío y recepción simultánea).
- Más rápida que I²C.
- Permite múltiples esclavos mediante líneas CS independientes.

---

## Análisis del código

El código realiza la lectura de temperatura desde un sensor **MAX6675**, que comunica vía SPI con un microcontrolador STM32.

### 1. Configuración SPI

Se inicializa el periférico SPI1 como maestro:
```c
hspi1.Init.Mode = SPI_MODE_MASTER;
hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_16;
```
- Se establece como **maestro**.
- Se usa **modo de transmisión de 8 bits**.
- Reloj en reposo bajo y muestreo en flanco de subida.
- Velocidad < 4.3 MHz como exige el MAX6675.

---

### 2. Control del pin CS

El pin **CS (Chip Select)** del MAX6675 está conectado al pin PA4. Se controla manualmente:
```c
#define MAX6675_CS_GPIO_PORT   GPIOA
#define MAX6675_CS_PIN         GPIO_PIN_4
```
Funciones asociadas:
```c
void MAX6675_Select(void);    // Baja CS (activo)
void MAX6675_Deselect(void);  // Sube CS (inactivo)
```

---

### 3. Lectura de datos del MAX6675

El sensor devuelve 16 bits. Se usa esta función:
```c
uint16_t MAX6675_ReadRaw(void);
```

Pasos:
1. Se baja el pin CS.
2. Se realiza una comunicación SPI (envío y recepción simultánea).
3. Se reensambla el valor crudo a partir de `rx[0]` y `rx[1]`.
4. Se vuelve a subir CS.

```c
value = ((uint16_t)rx[0] << 8) | rx[1];
```

---

### 4. Procesamiento del dato

Una vez recibido el dato crudo:
- Si el bit 2 (D2) está en 1, indica error (termopar desconectado).
- Si no hay error, se extraen los bits 3 a 14 y se multiplica por 0.25 para obtener °C:

```c
uint16_t temp_raw = (raw >> 3) & 0x0FFF;
temp = temp_raw * 0.25f;
```

---

### 5. Impresión por consola SWV (ITM)

Se redirige `printf` a la consola de depuración SWV (Serial Wire Viewer):
```c
int _write(int file, char *ptr, int len) {
    for (int i = 0; i < len; i++) {
        ITM_SendChar(*ptr++);
    }
    return len;
}
```

---

### 6. Ciclo principal (`while(1)`)

- Se llama cada segundo a `MAX6675_ReadRaw`.
- Se convierte la lectura a temperatura.
- Se imprime el valor crudo y la temperatura.
- Si hay error, se notifica.

---

## Conclusión

El código configura un canal SPI para comunicarse con el sensor MAX6675, obtiene los datos de temperatura en formato digital, los interpreta y los muestra a través del depurador. SPI se usa aquí por su velocidad y simplicidad para tareas donde se requiere adquirir datos periódicos desde un sensor dedicado.
