
# 🌡️ Lectura de Temperatura y Humedad con el Sensor SHT30 vía I2C (STM32)

Este proyecto implementa la comunicación con el sensor SHT30 utilizando el protocolo I2C en un microcontrolador STM32. El sistema mide temperatura y humedad, y muestra los resultados en el **ITM Console (Instrument Trace Macrocell)**.

---

## 📦 Requisitos

- Microcontrolador STM32 (ej. STM32F407)
- Sensor de temperatura y humedad SHT30
- Interfaz I2C (I2C1 configurado)
- Conexión SWD (para usar ITM/SWO)
- STM32CubeIDE
- ST-Link o depurador compatible con SWO

---

## 📁 Estructura del Proyecto

```
Core/
 ├── Inc/
 │    └── main.h       --> Definiciones y prototipos
 └── Src/
      └── main.c       --> Implementación principal
```

---

## ⚙️ Funcionamiento General

1. Inicializa el sistema, el reloj y el periférico I2C.
2. Comprueba si el sensor SHT30 está disponible.
3. Envía comando de medición al sensor.
4. Recibe datos crudos (RAW) de temperatura y humedad.
5. Verifica los CRC de ambos datos.
6. Convierte los valores RAW a grados Celsius y porcentaje de humedad.
7. Imprime los valores en consola ITM.

---

## 📄 Código principal (`main.c`)

```c
#include "main.h"
#include "stm32f4xx_hal.h"

#define SHT30_I2C_ADDR (0x44 << 1)

I2C_HandleTypeDef hi2c1;

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_I2C1_Init(void);

uint8_t SHT30_CalcCRC(uint8_t *data)
{
    uint8_t crc = 0xFF;
    for (int i = 0; i < 2; i++)
    {
        crc ^= data[i];
        for (int j = 0; j < 8; j++)
        {
            if (crc & 0x80)
                crc = (crc << 1) ^ 0x31;
            else
                crc <<= 1;
        }
    }
    return crc;
}

void ITM_Printf(const char *str)
{
    while (*str)
    {
        ITM_SendChar(*str++);
    }
}

HAL_StatusTypeDef SHT30_ReadTempHum(float *temperature, float *humidity)
{
    uint8_t cmd[] = {0x2C, 0x06};
    HAL_StatusTypeDef ret;

    ret = HAL_I2C_Master_Transmit(&hi2c1, SHT30_I2C_ADDR, cmd, 2, HAL_MAX_DELAY);
    if (ret != HAL_OK) return ret;

    HAL_Delay(15);

    uint8_t data[6];
    ret = HAL_I2C_Master_Receive(&hi2c1, SHT30_I2C_ADDR, data, 6, HAL_MAX_DELAY);
    if (ret != HAL_OK) return ret;

    if (SHT30_CalcCRC(data) != data[2] || SHT30_CalcCRC(&data[3]) != data[5])
        return HAL_ERROR;

    uint16_t rawTemp = (data[0] << 8) | data[1];
    uint16_t rawHum = (data[3] << 8) | data[4];

    *temperature = -45 + 175 * ((float)rawTemp / 65535.0f);
    *humidity = 100 * ((float)rawHum / 65535.0f);

    return HAL_OK;
}

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_I2C1_Init();

    char buffer[100];
    float temp, hum;

    if (HAL_I2C_IsDeviceReady(&hi2c1, SHT30_I2C_ADDR, 3, HAL_MAX_DELAY) == HAL_OK)
    {
        ITM_Printf("SHT30 detectado\r\n");
    }
    else
    {
        ITM_Printf("SHT30 no detectado\r\n");
        while (1);
    }

    while (1)
    {
        if (SHT30_ReadTempHum(&temp, &hum) == HAL_OK)
        {
            snprintf(buffer, sizeof(buffer), "Temp: %.2f C | Hum: %.2f %%RH\r\n", temp, hum);
            ITM_Printf(buffer);
        }
        else
        {
            ITM_Printf("Error leyendo SHT30 o CRC incorrecto\r\n");
        }

        HAL_Delay(2000);
    }
}
```

---

## 🧪 Ejemplo de salida en la consola ITM

```
SHT30 detectado
Temp: 24.68 C | Hum: 60.13 %RH
Temp: 24.72 C | Hum: 60.17 %RH
```

---

## 🛠️ Inicialización de Periféricos

### 🕓 `SystemClock_Config`

- Usa HSI como fuente de reloj.
- Configura AHB, APB1 y APB2.

### 🔌 `MX_GPIO_Init`

- Habilita el reloj GPIO para I2C.

### 📡 `MX_I2C1_Init`

```c
hi2c1.Instance = I2C1;
hi2c1.Init.ClockSpeed = 100000;
hi2c1.Init.DutyCycle = I2C_DUTYCYCLE_2;
hi2c1.Init.OwnAddress1 = 0;
hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
hi2c1.Init.OwnAddress2 = 0;
hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;
hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;
HAL_I2C_Init(&hi2c1);
```

---

## 🚨 Manejo de errores

- Verifica CRC para evitar errores de transmisión.
- Detiene ejecución si el sensor no responde.
- Informa errores de lectura o CRC.

---

## 🔄 Adaptaciones posibles

- Mostrar datos en pantalla OLED.
- Transmitir vía UART/Bluetooth.
- Uso de interrupciones/timers para muestreo.

---

## 👨‍💻 Autor

**Aníbal Yesid Romero Lambraño**
