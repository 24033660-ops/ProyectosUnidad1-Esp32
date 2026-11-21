# ⚙️ Práctica 4: Control de Posición Angular de Servomotor con Pulsadores (ESP32)

Sistema de posicionamiento angular de precisión ($0^\circ$ a $180^\circ$) para un servomotor mediante el microcontrolador **ESP32** y dos pulsadores físicos. Incorpora filtrado antirrebote (*debounce*) no bloqueante basado en `millis()`, aceleración continua al mantener presionados los botones, visualización en tiempo real en una pantalla **LCD I2C 16x2** y monitoreo por puerto serie.

---

## 📚 Librerías Necesarias

- **`Wire.h`**: Librería nativa incluida en el paquete base del ESP32 para la comunicación por bus I2C.
- **`ESP32Servo.h`**: Librería externa diseñada específicamente para generar señales PWM de control de servomotores mediante los temporizadores de hardware del ESP32 (autor: Kevin Harrington / John K. Bennett).
- **`LiquidCrystal_I2C.h`**: Librería externa para controlar pantallas LCD alfanuméricas mediante el módulo adaptador I2C `PCF8574` (autor: Frank de Brabander).

---

## 🧰 Componentes y Actuadores Utilizados

- **Microcontrolador:** ESP32 (DevKit V1 de 30 pines).
- **Actuador electromecánico:** 1x Servomotor estándar o micro-servo (ej. `SG90` o `MG996R`) de $180^\circ$.
- **Visualización:** 1x Pantalla **LCD 16x2** con módulo adaptador **I2C** (dirección `0x27`).
- **Interfaz de control:** 2x Pulsadores (*push buttons* normalmente abiertos - NA).
- **Alimentación y accesorios:** Fuente externa de 5 V CC (recomendada para el servo), protoboard y cables de conexión (*jumpers*).

---

## 🔌 Tabla de Conexiones (Pinout)

| Componente | Pin ESP32 | Modo / Bus | Conexión física y observaciones |
| :--- | :--- | :--- | :--- |
| **Servomotor (`Señal`)** | `GPIO 18` | `PWM (50 Hz)` | Cable de señal (naranja/amarillo) al `GPIO 18`; `VCC` (rojo) a `5V` y `GND` (marrón/negro) a `GND` común. |
| **Pulsador Subir (`BTN_UP`)** | `GPIO 32` | `INPUT_PULLUP` | Incrementa el ángulo (`+1°`); terminal opuesto directo a `GND`. |
| **Pulsador Bajar (`BTN_DOWN`)** | `GPIO 33` | `INPUT_PULLUP` | Decrementa el ángulo (`-1°`); terminal opuesto directo a `GND`. |
| **LCD I2C (`SDA`)** | `GPIO 21` | `I2C SDA` | Línea de datos I2C del módulo LCD (`0x27`). |
| **LCD I2C (`SCL`)** | `GPIO 22` | `I2C SCL` | Línea de reloj I2C del módulo LCD (`0x27`). |

---

## ⚙️ Funcionamiento del Sistema

1. **Generación de señal PWM calibrada:** El servomotor se configura a una frecuencia estándar de **`50 Hz`** (periodo de $20\,\text{ms}$) con anchos de pulso calibrados entre **`500 µs`** ($0^\circ$) y **`2400 µs`** ($180^\circ$) para aprovechar todo el recorrido mecánico del eje.
2. **Lectura no bloqueante con repetición rápida:** La estructura `Boton` administra los tiempos mediante `millis()` sin detener el flujo del programa con `delay()`:
   - **Antirrebote (*Debounce*):** Valida cada pulsación tras `25 ms` de estabilidad en la señal.
   - **Aceleración continua (*Auto-repeat*):** Si se mantiene presionado un botón durante más de `200 ms` (`HOLD_INICIO_MS`), el ángulo avanza automáticamente en pasos de $1^\circ$ cada **`20 ms`** (`HOLD_RE
