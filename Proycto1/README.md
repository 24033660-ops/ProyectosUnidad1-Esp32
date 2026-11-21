# 💡 Práctica 1: Control Digital de LED con Pulsadores (ESP32)

Implementación de entradas y salidas digitales (GPIO) en el microcontrolador **ESP32** para controlar el encendido y apagado independiente de un diodo LED mediante dos pulsadores físicos, incorporando detección de flancos de bajada, filtrado antirrebote (*debounce*) por software y diagnóstico en tiempo real a través del Monitor Serie.

---

## 📚 Librerías Necesarias

- **`Arduino.h`**: Librería base del entorno de desarrollo (incluida de forma nativa al seleccionar una placa ESP32 en Arduino IDE o PlatformIO). **No requiere instalar librerías externas.**

---

## 🧰 Componentes y Actuadores Utilizados

- **Microcontrolador:** ESP32 (DevKit V1 de 30 o 38 pines).
- **Actuador visual:** 1x Diodo LED estándar (5 mm).
- **Protección:** 1x Resistencia limitadora de corriente de $220\,\Omega$ a $330\,\Omega$ (1/4 W).
- **Entradas digitales:** 2x Pulsadores (*push buttons* normalmente abiertos - NA).
- **Accesorios:** Protoboard y cables de conexión (*jumpers*).

---

## 🔌 Tabla de Conexiones (Pinout)

| Componente | Pin ESP32 | Modo configurado | Conexión física |
| :--- | :--- | :--- | :--- |
| **LED (Ánodo `+`)** | `GPIO 23` | `OUTPUT` | Pin `GPIO 23` $\rightarrow$ Resistencia $220\,\Omega$ $\rightarrow$ Ánodo LED; Cátodo (`-`) a `GND`. |
| **Pulsador 1 (`SW1` - Encender)** | `GPIO 12` | `INPUT_PULLUP` | Un terminal al `GPIO 12` y el terminal opuesto directo a `GND`. |
| **Pulsador 2 (`SW2` - Apagar)** | `GPIO 13` | `INPUT_PULLUP` | Un terminal al `GPIO 13` y el terminal opuesto directo a `GND`. |

---

## ⚙️ Funcionamiento del Sistema

1. **Lógica inversa (`INPUT_PULLUP`):** Al habilitar las resistencias *pull-up* internas del ESP32, los pines `GPIO 12` y `GPIO 13` se mantienen en estado alto (`HIGH` / 3.3 V) mientras están en reposo y pasan a estado bajo (`LOW` / 0 V) al presionar el pulsador.
2. **Detección de flanco de bajada:** El programa compara el estado actual del botón con su estado anterior (`now == LOW && prev == HIGH`). De este modo, la acción se ejecuta una sola vez por cada pulsación y no de forma repetitiva si se deja el dedo presionado.
3. **Antirrebote (*Debounce*):** Incluye una pausa de `20 ms` al final del ciclo `loop()` para filtrar las oscilaciones mecánicas propias de los contactos metálicos del pulsador.
4. **Monitoreo Serial:** Configurado a **`115200 baudios`**, reporta en consola cada vez que se presiona el botón de encendido o apagado.

---

## 🛡️ Cuidados de Hardware y Precauciones Eléctricas

- **Protección del LED y del pin GPIO:** Nunca conectes el LED directamente entre el `GPIO 23` y `GND` sin resistencia. Los pines del ESP32 trabajan a **3.3 V** y tienen una corriente máxima recomendada de **12 mA** por pin; sin resistencia podrías quemar el LED o dañar permanentemente el puerto del microcontrolador.
- **Polaridad del LED:** Respeta la polaridad antes de energizar el circuito: la terminal más larga corresponde al **ánodo (`+`)** y el lado con el borde plano en el encapsulado corresponde al **cátodo (`-` / `GND`)**.
- **Conexión segura de pulsadores:** Dado que el código activa las resistencias *pull-up* internas, los botones deben cerrar circuito únicamente hacia **`GND`**. Nunca conectes un extremo del pulsador a `3.3V` o `5V` usando esta configuración, ya que provocarías un cortocircuito al presionarlo.
- **Precaución con el pin de arranque (`GPIO 12`):** En el ESP32, el `GPIO 12` es un *strapping pin* que define el voltaje de la memoria flash interna durante el encendido. Evita colocar resistencias *pull-up* externas conectadas a `3.3V` en este pin, pues si recibe nivel alto al arrancar impedirá que el microcontrolador inicie el programa.
- **Voltaje de operación:** El ESP32 **no es tolerante a 5 V** en sus pines lógicos. Asegúrate de alimentar tus circuitos de control exclusivamente con el pin `3V3` de la placa.
