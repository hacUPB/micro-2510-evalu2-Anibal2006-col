# ACTIVIDAD 3

## Programando una MEF en C para Antirrebote

### 🟡 Debounce (Anti-rebote)
Técnica para filtrar las fluctuaciones eléctricas (rebotes) que ocurren al presionar o soltar un pulsador. Consiste en esperar un intervalo fijo (por ejemplo, 30 ms) antes de confirmar un cambio de estado.

---

### 🔄 Diagrama de Estados
Representación gráfica de la MEF (Máquina de Estados Finitos) donde:
- Cada **nodo** representa un estado.
- Cada **flecha** representa una transición.
- Las flechas están **etiquetadas** con:
  - La **condición** de disparo.
  - (Opcionalmente) la **acción** o salida.

---

### 📋 Tabla de Transiciones
Vista tabular de las transiciones, donde:
- Cada fila indica:
  - El **estado actual**.
  - El **evento o condición**.
  - El **estado siguiente**.
  - (Opcionalmente) la **salida asociada**.

---

### ⏱️ SysTick
Temporizador interno del Cortex-M que genera una interrupción periódica.
Se usa normalmente para llevar cuenta del tiempo en milisegundos sin bloquear el programa.

#### Registros asociados:
- `SysTick->LOAD`: Valor inicial de la cuenta (por ejemplo, `SystemCoreClock/1000 - 1` para contar cada 1 ms).
- `SysTick->VAL`: Valor actual del contador. Escribir `0` lo reinicia.
- `SysTick->CTRL`: Controla el temporizador (habilitar, fuente de reloj, interrupción).

---

### 🛎️ SysTick_Handler()
Rutina de interrupción que se ejecuta cuando `SysTick` desborda.
Normalmente incrementa un **contador global de milisegundos**.

---

### 🕒 GetTick()
Función que retorna el valor del contador de milisegundos. 
Sirve para medir intervalos de tiempo sin bloqueos.

---

### 🧰 GPIO (General-Purpose I/O)
Puertos digitales del microcontrolador. Cada pin puede configurarse como:
- Entrada
- Salida
- Función alterna
- Analógico

#### Registros útiles:
- `RCC->AHB1ENR`: Habilita el reloj de los puertos GPIO (ejemplo: bit 0 para `GPIOA`).
- `GPIOx->MODER`: Define el modo de cada pin:
  - `00`: Entrada
  - `01`: Salida
  - `10`: Alterna
  - `11`: Analógico

---

### 🔘 Lectura de botón
Se hace leyendo el registro `GPIOA->IDR`:
- Si el bit correspondiente está en `1`, el botón está **presionado**.

---

### ⚙️ `enum`
Tipo de datos en C que define **constantes enteras con nombre**, usadas para representar los **estados** de la MEF, por ejemplo:
```c
enum { ESPERA, DEBOUNCE_PRESS, DEBOUNCE_RELEASE };


    Encapsular la MEF en archivos .c/.h.
    Evitar retardos bloqueantes (delay()).
    Usar volatile en contadores compartidos con ISR.
    Nombrar claramente estados con enum.
    Comentar y documentar cada función y estructura.



```c
/* USER CODE BEGIN Header */
/**
  ******************************************************************************
  * @file           : main.c
  * @brief          : Main program body
  ******************************************************************************
  * @attention
  *
  * Copyright (c) 2025 STMicroelectronics.
  * All rights reserved.
  *
  * This software is licensed under terms that can be found in the LICENSE file
  * in the root directory of this software component.
  * If no LICENSE file comes with this software, it is provided AS-IS.
  *
  ******************************************************************************
  */
#include "stm32f4xx.h"

// Pines de filas
#define ROW1 GPIO_PIN_1
#define ROW2 GPIO_PIN_3
#define ROW3 GPIO_PIN_5
#define ROW4 GPIO_PIN_7
#define ROW5 GPIO_PIN_5
#define ROW6 GPIO_PIN_1
#define ROW7 GPIO_PIN_8
#define ROW8 GPIO_PIN_10

// Pines de columnas
#define COL1 GPIO_PIN_0
#define COL2 GPIO_PIN_2
#define COL3 GPIO_PIN_4
#define COL4 GPIO_PIN_6
#define COL5 GPIO_PIN_4
#define COL6 GPIO_PIN_0
#define COL7 GPIO_PIN_7
#define COL8 GPIO_PIN_9

// Puertos
#define ROW_PORT_1 GPIOA
#define ROW_PORT_2 GPIOC
#define ROW_PORT_3 GPIOB
#define ROW_PORT_4 GPIOE

#define COL_PORT_1 GPIOA
#define COL_PORT_2 GPIOC
#define COL_PORT_3 GPIOB
#define COL_PORT_4 GPIOE

// Frames de ejemplo
uint8_t frame1[8] = {
    0b00000000,
    0b01100110,
    0b11111111,
    0b11111111,
    0b01111110,
    0b00111100,
    0b00011000,
    0b00000000
};

uint8_t frame2[8] = {
    0b00111111,
    0b01111111,
    0b11000011,
    0b11000011,
    0b11000011,
    0b11000011,
    0b01111111,
    0b00111111
};

uint8_t frame3[8] = {
    0b01111111,
    0b01111111,
    0b00000011,
    0b00111111,
    0b00111111,
    0b00000011,
    0b00000011,
    0b00000011
};

uint8_t* frames[] = {frame1, frame2, frame3};
const uint8_t total_frames = 3;
uint8_t current_frame = 0;

// Pines
GPIO_TypeDef *row_ports[] = {ROW_PORT_1, ROW_PORT_1, ROW_PORT_1, ROW_PORT_1, ROW_PORT_2, ROW_PORT_3, ROW_PORT_4, ROW_PORT_4};
uint16_t row_pins[]       = {ROW1, ROW2, ROW3, ROW4, ROW5, ROW6, ROW7, ROW8};

GPIO_TypeDef *col_ports[] = {COL_PORT_1, COL_PORT_1, COL_PORT_1, COL_PORT_1, COL_PORT_2, COL_PORT_3, COL_PORT_4, COL_PORT_4};
uint16_t col_pins[]       = {COL1, COL2, COL3, COL4, COL5, COL6, COL7, COL8};

// Máquina de estados
typedef enum {
    ESTADO_FILA_0,
    ESTADO_FILA_1,
    ESTADO_FILA_2,
    ESTADO_FILA_3,
    ESTADO_FILA_4,
    ESTADO_FILA_5,
    ESTADO_FILA_6,
    ESTADO_FILA_7
} EstadoMatriz;

EstadoMatriz estado_actual = ESTADO_FILA_0;

uint32_t last_state_change_time = 0;
uint32_t last_frame_change_time = 0;
uint8_t filas_mostradas = 0;

void apagar_filas(void) {
    for (int i = 0; i < 8; i++) {
        HAL_GPIO_WritePin(row_ports[i], row_pins[i], GPIO_PIN_SET);
    }
}

void mostrar_fila(uint8_t fila, uint8_t *pattern) {
    apagar_filas();
    HAL_GPIO_WritePin(row_ports[fila], row_pins[fila], GPIO_PIN_RESET);
    for (int col = 0; col < 8; col++) {
        HAL_GPIO_WritePin(col_ports[col], col_pins[col], (pattern[fila] & (1 << col)) ? GPIO_PIN_SET : GPIO_PIN_RESET);
    }
}

void matriz_update(void) {
    uint32_t ahora = HAL_GetTick();

    switch (estado_actual) {
        case ESTADO_FILA_0:
            if (ahora - last_state_change_time >= 2) {
                last_state_change_time = ahora;
                mostrar_fila(0, frames[current_frame]);
                estado_actual = ESTADO_FILA_1;
            }
            break;
        // ... repetir para ESTADO_FILA_1 a ESTADO_FILA_7 ...
        case ESTADO_FILA_7:
            if (ahora - last_state_change_time >= 2) {
                last_state_change_time = ahora;
                mostrar_fila(7, frames[current_frame]);
                estado_actual = ESTADO_FILA_0;

                filas_mostradas++;
                if (filas_mostradas >= 1) {
                    filas_mostradas = 0;
                    if ((ahora - last_frame_change_time) >= 4000) {
                        last_frame_change_time = ahora;
                        current_frame = (current_frame + 1) % total_frames;
                    }
                }
            }
            break;
    }
}

// Inicialización GPIO
void MX_GPIO_Init(void) {
    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOB_CLK_ENABLE();
    __HAL_RCC_GPIOC_CLK_ENABLE();
    __HAL_RCC_GPIOE_CLK_ENABLE();

    GPIO_InitTypeDef GPIO_InitStruct = {0};
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;

    // Filas
    GPIO_InitStruct.Pin = ROW1 | ROW2 | ROW3 | ROW4;
    HAL_GPIO_Init(ROW_PORT_1, &GPIO_InitStruct);
    GPIO_InitStruct.Pin = ROW5;
    HAL_GPIO_Init(ROW_PORT_2, &GPIO_InitStruct);
    GPIO_InitStruct.Pin = ROW6;
    HAL_GPIO_Init(ROW_PORT_3, &GPIO_InitStruct);
    GPIO_InitStruct.Pin = ROW7 | ROW8;
    HAL_GPIO_Init(ROW_PORT_4, &GPIO_InitStruct);

    // Columnas
    GPIO_InitStruct.Pin = COL1 | COL2 | COL3 | COL4;
    HAL_GPIO_Init(COL_PORT_1, &GPIO_InitStruct);
    GPIO_InitStruct.Pin = COL5;
    HAL_GPIO_Init(COL_PORT_2, &GPIO_InitStruct);
    GPIO_InitStruct.Pin = COL6;
    HAL_GPIO_Init(COL_PORT_3, &GPIO_InitStruct);
    GPIO_InitStruct.Pin = COL7 | COL8;
    HAL_GPIO_Init(COL_PORT_4, &GPIO_InitStruct);
}

void SystemClock_Config(void);  // Implementar según tu plataforma

int main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();

    while (1) {
        matriz_update();
    }
}

void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  __HAL_RCC_PWR_CLK_ENABLE();
  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_NONE;
  HAL_RCC_OscConfig(&RCC_OscInitStruct);
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_HSI;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;
  HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_0);
}

void Error_Handler(void) {
    __disable_irq();
    while (1) {}
}

#ifdef  USE_FULL_ASSERT
void assert_failed(uint8_t *file, uint32_t line) {}
#endif

La multiplexación es una técnica que permite controlar múltiples dispositivos o líneas con un número reducido de pines GPIO, aprovechando la alta velocidad de refresco de los microcontroladores para que el usuario perciba un funcionamiento continuo.
Problema:
    Sin multiplexación, cada elemento (LED, tecla) requeriría un pin dedicado, lo que rápidamente agota los recursos de E/S de un microcontrolador.
Idea clave:
    En una matriz (por ejemplo, LEDs 8×8 o teclado 4×4) se agrupan filas y columnas. Se activan secuencialmente unas pocas líneas (p. ej., 8 filas + 8 columnas) en lugar de todas a la vez.
Cómo funciona:
    Configurar pines: asignar un grupo de pines a las filas y otro a las columnas.
    Salida–Salida (LEDs): todas las líneas en modo salida; se pone alta la fila deseada (cátodo común) y se calcula qué columnas deben encenderse para ese “frame”.
    Salida–Entrada (teclado): filas como salidas, columnas como entradas con resistencias de pull-up; se pone a nivel bajo una fila tras otra y se leen las columnas para detectar la tecla presionada.
    Ciclo rápido: se repite este barrido fila por fila (o columna por columna) a una frecuencia suficiente para que el ojo humano (o el dedo) perciba un estado “estable”.
Beneficios:
    Ahorra hasta un 50 % de pines (p. ej., 8×8 LEDs con 16 pines en lugar de 64).
    Mantiene bajo consumo y simplicidad de cableado.
    Esencial en sistemas embebidos y dispositivos IoT con recursos limitados.
Consideraciones:
    La velocidad de barrido debe ser mayor que la frecuencia de refresco mínima perceptible (aprox. 50–100 Hz).
    En teclados se usan resistencias de pull-up para evitar líneas flotantes y aplicar debounce sobre cada detección.
    El microcontrolador debe alternar entre “activar” una fila y “leer” las columnas, o viceversa, sin bloquear el resto de tareas.
