# 🧠 Máquinas de Estados Finitos (MEF)

Una **máquina de estados finitos** es un modelo matemático y gráfico que representa sistemas cuyo comportamiento depende de una secuencia de eventos.

## 🧩 Elementos clave

- **Estados**: Conjunto finito de “situaciones” en las que el sistema puede encontrarse.
- **Entradas**: Eventos o señales externas que disparan cambios.
- **Transiciones**: Reglas que, según el estado actual y (en el caso de Mealy) la entrada, determinan el siguiente estado.
- **Salidas**: Acciones o señales generadas, que pueden depender:
  - Solo del estado (Máquina de **Moore**).
  - Del estado y la entrada (Máquina de **Mealy**).

---

## 🟢 Máquina de Moore

En una **Máquina de Moore**, las salidas dependen **solo del estado actual**, sin considerar las entradas en el instante de transición.

### ✳️ Características:

- **Salidas ligadas al estado**:  
  Cada estado tiene asociadas una o más salidas fijas.  
  Al entrar a un estado, se activan inmediatamente las salidas correspondientes.

- **Transición basada en la entrada**:  
  Las transiciones se disparan cuando se cumple una condición sobre la entrada,  
  pero las salidas no cambian hasta ingresar al nuevo estado.

### ✅ Ventajas:
- **Salidas estables**: No parpadean si la entrada cambia bruscamente.
- **Diseño más sencillo**: Útil cuando no se requiere reacción instantánea a la entrada.

### ⚠️ Inconvenientes:
- **Latencia en la respuesta**: La salida cambia solo al entrar al nuevo estado.

---

## 🔵 Máquina de Mealy

Una **Máquina de Mealy** genera salidas que dependen tanto del estado actual como de las entradas **en el momento de la transición**.

### ✳️ Características:

- **Salidas ligadas a la transición**:  
  Cada flecha de transición se anota como:  
  `condición_entrada / salida`  
  La salida puede cambiar **inmediatamente** al cumplirse la condición, sin esperar al siguiente estado.

- **Transición basada en estado + entrada**:  
  El siguiente estado se determina como en cualquier FSM,  
  pero la **salida se genera al evaluar la entrada**, sin cambiar de estado.

### ✅ Ventajas:
- **Respuesta más rápida**: Las salidas reaccionan al instante ante cambios de entrada.
- **Menor número de estados**: Puede simplificar el diseño general.

### ⚠️ Inconvenientes:
- **Diseño más complejo**:  
  Las salidas pueden volverse menos predecibles, ya que cambian dentro de las transiciones.

---


Diferencias:

| Característica            | Máquina de Moore                          | Máquina de Mealy                                        |
|---------------------------|-------------------------------------------|---------------------------------------------------------|
| **Dependencia de la salida** | Solo del estado actual                    | Del estado actual **y** de la entrada                   |
| **Lugar de la salida**    | Asociada a cada estado                    | Asociada a cada transición                              |
| **Latencia en la respuesta** | Mayor: la salida cambia al entrar en el siguiente estado | Menor: la salida puede cambiar inmediatamente           |
| **Número de estados**     | Suele ser mayor                           | Suele ser menor                                          |
| **Estabilidad de salidas**| Alta                                      | Menor                                                   |
| **Complejidad de diseño** | Más simple                                | Más complejo                                            |
| **Casos de uso típicos**  | Salidas estables (semáforos, displays)    | Respuesta rápida (controles interactivos, protocolos)   |
