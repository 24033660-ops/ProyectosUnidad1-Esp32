# 💧 Práctica 2: Medición y Control de Nivel de Agua con Sensor Ultrasónico (ESP32)

Sistema de monitoreo y control de nivel de líquido en un tanque mediante el microcontrolador **ESP32** y un sensor ultrasónico **HC-SR04**. El programa convierte la distancia medida en un porcentaje calibrado ($0\%$ a $100\%$), despliega el nivel y el estado operativo en una pantalla **LCD I2C 16x2**, y activa indicadores LED mediante una banda de histéresis para señalar el llenado o el nivel máximo.

---

## 📚 Librerías Necesarias

- **`Wire.h`**: Librería nativa para la comunicación por bus I2C (incluida en el paquete base de ESP32).
- **`LiquidCrystal_I2C.h`**: Librería externa para controlar pantallas LCD alfanuméricas mediante módulo adaptador I2C (`PCF8574`). Se instala desde el *Gestor de Librerías* de Arduino IDE (autor: Frank de Brabander) o mediante PlatformIO.

---

## 🧰 Componentes, Sensores y Actuadores Utilizados

- **Microcontrolador:** ESP32 (DevKit V1 de 30 pines).
- **Sensor de nivel:** 1x Sensor ultrasónico **HC-SR04** (rango de $2\,\text{cm}$ a $400\,\text{cm}$).
- **Visualización:** 1x Pantalla **LCD 16x2** con módulo adaptador **I2C** (dirección `0x27`).
- **Actuadores visuales:** 2x Diodos LED (indicador de llenado y de nivel alto).
- **Resistencias:**
  - 1x Resistencia de $1\,\text{k}\Omega$ y 1x Resistencia de $2\,\text{k}\Omega$ (para el divisor de voltaje del pin `ECHO`).
  - 2x Resistencias de $220\,\Omega$ a $330\,\Omega$ (limitadoras de corriente para los LEDs).
- **Entradas auxiliares:** 3x Pulsadores (*push buttons* NA) conectados a tierra como base de hardware.

---

## 🔌 Tabla de Conexiones (Pinout)

| Componente | Pin ESP32 | Modo / Bus | Conexión física y observaciones |
| :--- | :--- | :--- | :--- |
| **HC-SR04 (`TRIG`)** | `GPIO 5` | `OUTPUT` | Conexión directa al pin `TRIG` del sensor. |
| **HC-SR04 (`ECHO`)** | `GPIO 18` | `INPUT` | **Con divisor de voltaje:** `ECHO` $\rightarrow$ $1\,\text{k}\Omega$ $\rightarrow$ `GPIO 18` $\rightarrow$ $2\,\text{k}\Omega$ $\rightarrow$ `GND`. |
| **LCD I2C (`SDA`)** | `GPIO 21` | `I2C SDA` | Línea de datos I2C del módulo LCD (`0x27`). |
| **LCD I2C (`SCL`)** | `GPIO 22` | `I2C SCL` | Línea de reloj I2C del módulo LCD (`0x27`). |
| **LED 1 (Subiendo agua)** | `GPIO 26` | `OUTPUT` | `GPIO 26` $\rightarrow$ Resistencia $220\,\Omega$ $\rightarrow$ Ánodo (`+`); Cátodo a `GND`. |
| **LED 2 (Nivel alto)** | `GPIO 27` | `OUTPUT` | `GPIO 27` $\rightarrow$ Resistencia $220\,\Omega$ $\rightarrow$ Ánodo (`+`); Cátodo a `GND`. |
| **Pulsadores (`BTN1`–`BTN3`)** | `GPIO 32, 33, 25` | `INPUT_PULLUP` | Configurados en `setup()` con resistencia interna hacia `GND` para expansión. |

---

## ⚙️ Funcionamiento y Calibración del Sistema

1. **Medición ultrasónica:** Cada `300 ms`, el ESP32 envía un pulso de disparo de $10\,\mu\text{s}$ por `GPIO 5` y mide el tiempo de retorno del eco mediante `pulseIn(ECHO_PIN, HIGH, 30000)` (con *timeout* de $30\,\text{ms}$ para evitar bloqueos si no hay rebote). La distancia se calcula usando la velocidad del sonido:
   $$\text{Distancia (cm)} = \frac{\text{tiempo } (\mu\text{s}) \times 0.0343}{2}$$
2. **Calibración lineal del tanque:** El código mapea las dimensiones físicas del recipiente a porcentaje útil ($0\%$ a $100\%$), donde **`19.0 cm`** representa el tanque vacío (`DIST_VACIO`) y **`7.0 cm`** representa el tanque lleno (`DIST_LLENO`):
   $$\text{Porcentaje} = \frac{19.0 - \text{Distancia}}{19.0 - 7.0} \times 100$$
3. **Control por histéresis (evita parpadeos por oleaje):**
   - **Nivel bajo ($\le 20\%$):** Enciende **LED 1** (`GPIO 26`) y apaga **LED 2** (`Subiendo agua`).
   - **Nivel alto ($\ge 90\%$):** Apaga **LED 1** y enciende **LED 2** (`GPIO 27`, `Nivel alto`).
   - **Zona intermedia ($21\%$ a $89\%$):** Conserva el último estado activo (`Esperando...` si aún no alcanza un umbral), evitando conmutaciones erráticas por pequeñas ondas en la superficie del agua.
4. **Diagnóstico Dual:** Muestra el porcentaje y estado en la pantalla LCD (`0x27`) y envía la lectura detallada en centímetros y porcentaje al Monitor Serie a **`115200 baudios`**.

---

## 🛡️ Cuidados de Sensores y Precauciones Eléctricas

- **Divisor de voltaje obligatorio en `ECHO`:** El sensor HC-SR04 se alimenta a **5 V** (`VIN`) y devuelve una señal de **5 V** en el pin `ECHO`. Dado que los pines del ESP32 soportan únicamente **3.3 V**, el divisor con resistencias de **$1\,\text{k}\Omega$ y $2\,\text{k}\Omega$** reduce el pulso a $\approx 3.33\,\text{V}$, evitando dañar el `GPIO 18`.
- **Protección contra humedad y condensación:** El sensor HC-SR04 estándar **no es sumergible ni impermeable**. Debe instalarse en la tapa superior del tanque mirando perpendicularmente hacia el agua, evitando salpicaduras o condensación excesiva en las cápsulas emisora y receptora.
- **Respeto a la zona muerta del sensor:** Todo sensor ultrasónico tiene una zona ciega mínima de $2\,\text{cm}$ a $4\,\text{cm}$ donde las lecturas son erráticas. El umbral `DIST_LLENO = 7.0 cm` garantiza que el agua nunca invada la zona ciega ni toque el sensor.
- **Alineación mecánica:** Monta el sensor completamente horizontal respecto a la superficie del líquido; inclinaciones mayores a $15^\circ$ desvían el rebote acústico y provocan lecturas fuera de rango (`---`).
- **Ajuste de contraste del LCD:** Si la pantalla LCD enciende su luz de fondo pero no muestra caracteres, gira suavemente el potenciómetro azul en la parte trasera del módulo I2C hasta que los bloques de texto sean nítidos.
