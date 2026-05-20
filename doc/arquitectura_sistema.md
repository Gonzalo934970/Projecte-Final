### 1. Introducción y Objetivos

El objetivo principal de este sistema es mitigar la gestión ineficiente del agua en la granja GreenFarm mediante la automatización del riego basada en datos en tiempo real. Se busca reducir el consumo hídrico innecesario, asegurar los niveles óptimos de humedad para los cultivos y prevenir pérdidas por fugas.

---

### 2. Arquitectura General (Esquema de Bloques)

El sistema se compone de tres capas principales: Capa de Captura y Actuación (Hardware), Capa de Comunicación (Red), y Capa de Aplicación (Cloud & Dashboard).

```
[ Capa de Captura y Actuación ]
     │   (Lectura analógica / digital)
     ▼
 ┌──────────────────────────────────────┐
 │ Microcontrolador (Ej: ESP32 / NodeMCU)│
 └──────────────────┬───────────────────┘
                    │
                    │ (Wi-Fi - Protocolo MQTT)
                    ▼
[ Capa de Comunicación / Broker ]
 ┌──────────────────────────────────────┐
 │ Broker MQTT (Ej: HiveMQ / Mosquitto) │
 └──────────────────┬───────────────────┘
                    │
                    │ (Suscripción de datos)
                    ▼
[ Capa de Aplicación y Usuario ]
 ┌──────────────────────────────────────┐
 │ Dashboard (Ej: Node-RED / Adafruit)  │
 └──────────────────────────────────────┘

```

---

### 3. Especificación de Hardware (Capa de Captura y Actuación)

Para este despliegue se han seleccionado los siguientes componentes tecnológicos:

| Componente | Función Técnica | Justificación |
| --- | --- | --- |
| **Microcontrolador ESP32** | Cerebro del sistema, procesa los datos de los sensores y controla los actuadores. | Cuenta con conectividad Wi-Fi integrada de bajo coste y suficientes pines analógicos/digitales. |
| **Sensor de Humedad de Suelo (Capacitivo v1.2)** | Mide el porcentaje de agua retenida en la tierra. | Se elige un modelo capacitivo en lugar de uno resistivo para evitar la corrosión prematura del sensor al estar enterrado. |
| **Sensor de Flujo de Agua (Caudalímetro YF-S201)** | Mide los litros de agua por minuto que pasan por la tubería principal. | Permite auditar el consumo real del cliente y detectar si hay un flujo de agua cuando el riego está apagado (fugas). |
| **Módulo Relé de 5V/12V** | Actúa como un interruptor electrónico controlado por el ESP32. | Permite aislar la corriente del microcontrolador de la potencia requerida por la válvula de agua. |
| **Electroválvula de Solenoide (12V)** | Abre o cierra el paso físico del agua de riego de forma automática. | Es un estándar industrial fiable para el control de fluidos a baja presión. |

---

### 4. Capa de Comunicación y Red

* **Protocolo de Red:** Wi-Fi (802.11 b/g/n) conectado a la red local de la granja.
* **Protocolo de Mensajería:** **MQTT (Message Queuing Telemetry Transport)**. Se selecciona por su ligereza, bajo consumo de ancho de banda y excelente rendimiento en dispositivos IoT con recursos limitados.
* **Tópicos (Topics) definidos:**
* `greenfarm/granja1/humedad` -> Envío de datos del sensor (Payload: valor porcentual de 0 a 100).
* `greenfarm/granja1/caudal` -> Envío de consumo de agua (Payload: Litros/min).
* `greenfarm/granja1/riego/estado` -> Estado actual de la válvula (Payload: `ON` / `OFF`).
* `greenfarm/granja1/riego/comando` -> Recepción de órdenes desde el dashboard (Payload: `OPEN` / `CLOSE`).



---

### 5. Lógica de Control (Algoritmo de Riego)

El microcontrolador ejecutará una lógica basada en umbrales (control Todo/Nada con histéresis) para evitar activaciones repetitivas:

1. **Condición de Activación:** Si la humedad del suelo cae por debajo del **35%**, el ESP32 activará el relé para abrir la electroválvula.
2. **Condición de Parada:** El riego se detendrá inmediatamente cuando ocurra cualquiera de estos dos eventos:
* La humedad del suelo alcance el **75%**.
* El tiempo máximo de riego de seguridad (ej. 15 minutos) se haya cumplido (para evitar inundaciones si falla el sensor).


3. **Seguridad contra Fugas:** Si el sensor de flujo detecta paso de agua (`caudal > 0`) pero el estado del riego es `OFF`, el sistema enviará una alerta crítica de **Fuga Detectada**.

---

### 6. Capa de Aplicación (Interfaz del Cliente)

El cliente final interactuará con el sistema a través de una plataforma IoT en la nube. La interfaz constará de:

* Un gráfico lineal histórico de la humedad de las últimas 24 horas.
* Un indicador numérico (Gauge) del consumo diario acumulado de agua.
* Un interruptor manual de emergencia para forzar el encendido/apagado del riego desde el teléfono móvil.

---

### Conclusión del Issue

Este diseño de arquitectura servirá como guía de referencia para todo el equipo de ingeniería durante las fases de montaje de hardware (Fase 2) y programación del código fuente (Fase 3).
