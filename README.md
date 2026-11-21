# ⚡ ProyectosUnidad1-Esp32

Repositorio con las prácticas fundamentales de entradas/salidas digitales, sensores, actuadores y sistemas de control desarrolladas con microcontroladores (ESP32). Dentro de la carpeta de cada práctica se adjuntan los 📄 **documentos de reporte**, el 💻 **código fuente en uso** y el 🔌 **diagrama de conexiones** correspondiente.

## 📂 Contenido del repositorio

- 💡 **proyecto 1:** Control digital de encendido y apagado de un LED mediante dos pulsadores configurados con resistencias *pull-up* internas (`INPUT_PULLUP`), detección de flancos, antirrebote (*debounce*) y monitoreo de eventos por puerto serie.
- 💧 **proyecto 2:** Sistema de medición y control de nivel de agua en un tanque utilizando un sensor ultrasónico (HC-SR04), conversión de distancia calibrada a porcentaje (0 % a 100 %), pantalla LCD I2C 16x2 y dos LEDs indicadores con histéresis para señalar nivel bajo/llenado (≤ 20 %) y nivel alto (≥ 90 %).
- 🌡️ **proyecto 3:** Control térmico ON/OFF con histéresis utilizando un sensor digital DS18B20, tres pulsadores con antirrebote para ajustar los umbrales mínimo y máximo (`±1 °C`), pantalla LCD I2C 16x2 para visualizar la temperatura y el estado del ciclo (`[CAL]`/`[ENF]`), y dos relés para alternar entre un foco calefactor y una celda Peltier.
- ⚙️ **proyecto 4:** Control de posición angular de un servomotor (0° a 180°) mediante dos pulsadores con antirrebote y repetición rápida continua al mantenerlos presionados, visualizando el ángulo en tiempo real en una pantalla LCD I2C 16x2.
