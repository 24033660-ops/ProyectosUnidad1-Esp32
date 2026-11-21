# 🌡️ Práctica 3: Control Térmico ON/OFF con Histéresis (ESP32)

Sistema de control de temperatura en lazo cerrado con histéresis desarrollado con el microcontrolador **ESP32**. Utiliza un sensor digital **DS18B20** para medir la temperatura en tiempo real, **tres pulsadores** para configurar dinámicamente los umbrales mínimo y máximo, una pantalla **LCD I2C 16x2** para visualizar el estado del proceso y un **módulo de 2 relés** para alternar entre una etapa de calentamiento (foco incandescente) y una etapa de enfriamiento (celda Peltier).

---

## 📚 Librerías Necesarias

- **`Arduino.h` y `Wire.h`**: Librerías nativas incluidas en el paquete base del ESP32 para funciones generales y comunicación I2C.
- **`OneWire.h`**: Librería externa necesaria para gestionar el protocolo de comunicación de un solo hilo (*1-Wire*) del sensor DS18B20 (autor: Paul Stoffregen).
- **`DallasTemperature.h`**: Librería externa que facilita la lectura en grados Celsius y la configuración de resolución del sensor DS18B20 (autor: Miles Burton).
- **`LiquidCrystal_I2C.h`**: Librería externa para controlar la pantalla LCD alfanumérica mediante el adaptador I2C `PCF8574` (autor: Frank de Brabander).

---

## 🧰 Componentes, Sensores y Actuadores Utilizados

- **Microcontrolador:** ESP32 (DevKit V1 de 30 pines).
- **Sensor de temperatura:** 1x Sensor digital **DS18B20** (encapsulado TO-92 o sonda impermeable) + 1x Resistencia *pull-up* de $4.7\,\text{k}\Omega$.
- **Visualización:** 1x Pantalla **LCD 16x2** con módulo adaptador **I2C** (dirección `0x27`).
- **Interfaz de control:** 3x Pulsadores (*push buttons* normalmente abiertos - NA).
- **Etapa de potencia y actuadores:**
  - 1x Módulo de **2 relés** de 5 V con optoacopladores (lógica *Active LOW*).
  - 1x Foco incandescente (carga resistiva de calentamiento en CA) con socket y clavija.
  - 1x Celda Peltier (ej. `TEC1-12706` a 12 V CC) con disipador de aluminio, ventilador y pasta térmica.
- **Fuentes de alimentación:** Fuente de 5 V para el ESP32/relés y fuente dedicada de 12 V CC (mínimo 5 A) para la celda Peltier.

---

## 🔌 Tabla de Conexiones (Pinout)

| Componente | Pin ESP32 | Modo / Bus | Conexión física y observaciones |
| :--- | :--- | :--- | :--- |
| **DS18B20 (`DATA`)** | `GPIO 4` | `1-Wire` | Pin central (`DATA`) al `GPIO 4` con resistencia de **$4.7\,\text{k}\Omega$** hacia `3.3V`. |
| **LCD I2C (`SDA`)** | `GPIO 21` | `I2C SDA` | Línea de datos I2C del módulo LCD (`0x27`). |
| **LCD I2C (`SCL`)** | `GPIO 22` | `I2C SCL` | Línea de reloj I2C del módulo LCD (`0x27`). |
| **Botón 1 (`+1 °C`)** | `GPIO 32` | `INPUT_PULLUP` | Incrementa el umbral seleccionado; terminal opuesto a `GND`. |
| **Botón 2 (`-1 °C`)** | `GPIO 33` | `INPUT_PULLUP` | Decrementa el umbral seleccionado; terminal opuesto a `GND`. |
| **Botón 3 (`Min / Max`)** | `GPIO 25` | `INPUT_PULLUP` | Alterna el cursor `*` entre editar `Min` o `Max`; terminal opuesto a `GND`. |
| **Relé 1 (Foco)** | `GPIO 2` | `OUTPUT` | Señal `IN1` del módulo de relés (*Active LOW*: `LOW` = encendido). |
| **Relé 2 (Peltier)** | `GPIO 15` | `OUTPUT` | Señal `IN2` del módulo de relés (*Active LOW*: `LOW` = encendido). |

---

## ⚙️ Funcionamiento del Sistema

1. **Adquisición de temperatura no bloqueante:** El sensor **DS18B20** se configura a resolución de **9 bits** ($0.5\,^\circ\text{C}$ de paso, conversión rápida) y modo asíncrono (`setWaitForConversion(false)`), leyendo la temperatura cada `1000 ms` sin detener la respuesta de los botones.
2. **Ajuste interactivo de umbrales:** Mediante los tres pulsadores (con filtrado antirrebote por software de `180 ms`), el usuario selecciona con el **Botón 3** (`GPIO 25`) el límite a modificar (indicado con un asterisco `*` en la pantalla LCD) y ajusta su valor en pasos de $\pm 1\,^\circ\text{C}$ con los **Botones 1 y 2**. El código impide que `tempMin` iguale o supere a `tempMax`.
3. **Lógica de control por histéresis:**
   - **Estado `CALENTANDO` (`[CAL]`):** Mantiene encendido el **Foco** (`GPIO 2` en `LOW`) y apagada la **Peltier** hasta que $T_{\text{actual}} \ge T_{\text{max}}$ ($60\,^\circ\text{C}$ por defecto).
   - **Estado `ENFRIANDO` (`[ENF]`):** Al alcanzar el límite superior, apaga el foco y activa la **Celda Peltier** (`GPIO 15` en `LOW`) hasta que la temperatura descienda a $T_{\text{actual}} \le T_{\text{min}}$ ($30\,^\circ\text{C}$ por defecto), reiniciando el ciclo.
4. **Monitoreo en LCD y Puerto Serie:** La pantalla LCD muestra en la fila superior la temperatura y el estado activo (`[CAL]` / `[ENF]`), y en la inferior los umbrales `*Min` y `Max`. Simultáneamente, envía el registro completo al Monitor Serie a **`115200 baudios`**.

---

## 🛡️ Cuidados de Sensores, Actuadores y Precauciones Eléctricas

- **Seguridad con Corriente Alterna (110 V / 220 V CA):** El circuito del foco maneja voltaje de red eléctrico potencialmente mortal. Aísla todas las terminales de CA con cinta aislante o termofit, mantén el cableado de potencia físicamente separado de la protoboard del ESP32 y **nunca toques la parte inferior de la placa de relés** mientras esté conectada a la toma de corriente.
- **Disipación térmica obligatoria en la Celda Peltier:** Nunca energices una celda Peltier sin un **disipador de calor con ventilador y pasta térmica** acoplados en su cara caliente. Sin disipación, la celda se sobrecalienta en pocos segundos, pierde su capacidad de enfriamiento y sufre daños permanentes en sus uniones semiconductoras. Además, aliméntala con una **fuente externa de 12 V CC**, nunca desde el ESP32.
- **Resistencia *pull-up* y polaridad del DS18B20:** Es indispensable colocar la resistencia de **$4.7\,\text{k}\Omega$** entre el pin `DATA` (`GPIO 4`) y `3.3V` para evitar lecturas erróneas (`-127 °C`). Verifica cuidadosamente los pines `GND`, `DATA` y `VDD` antes de energizar: invertir la polaridad calienta el encapsulado al instante y destruye el sensor.
- **Precaución con pines de arranque (`GPIO 2` y `GPIO 15`):** Ambos pines son *strapping pins* en el ESP32. Si al intentar cargar un programa nuevo desde la computadora aparece el error *"Failed to connect to ESP32"*, desconecta temporalmente el cable del `GPIO 2` mientras se sube el código, ya que algunos módulos de relés *Active LOW* mantienen el `GPIO 2` en estado alto e impiden entrar en modo *flash*.
- **Alimentación del módulo de relés:** Las bobinas de los relés requieren **5 V** (`VIN` del ESP32 o fuente externa de 5 V con `GND` común) para conmutar mecánicamente con firmeza, mientras que las entradas `IN1` e `IN2` responden correctamente a los pulsos de **3.3 V** del microcontrolador.
