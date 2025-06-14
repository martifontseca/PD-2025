# Práctica 1: Blink con ESP32

---

## Objetivo

Esta práctica tiene como finalidad lograr el parpadeo periódico de un LED mediante el uso de un ESP32. Para ello, se utiliza la salida serie como herramienta 
de depuración y se exploran diversas técnicas: desde funciones estándar de Arduino hasta el acceso directo a registros y distintos métodos de temporización.
Al final, se evalúa el rendimiento del microcontrolador.

---

## Microcontrolador: ESP32

### Imagen del ESP32

![ESP32](https://ae04.alicdn.com/kf/S8dee2f4cafc344e1b57ebc21ad5c11a4P.jpg?fit=600%2C600&ssl=1)

### Distribución de pines

![Pines ESP32](https://ae04.alicdn.com/kf/S61a9f7eb6ad3487ca95acc2f410157a35.jpg?resize=966%2C574&ssl=1)

---

##  Descripción de la funcionalidad

- Iniciar pin de LED como salida
- Bucle infinito:
  - Encender LED
  - Enviar mensaje "ON" por el puerto serie
  - Esperar 500 o 1000 ms
  - Apagar LED
  - Enviar mensaje "OFF" por el puerto serie
  - Esperar 500 o 1000 ms# PRACTICA 1  :  BLINK

El objeticvo de la practica es producir el parpadeo periodico de un led. 
Se utilizara la salida serie  para depurar  el programa 

El microcontrolador que utilizaremos es el ya comentado en la introducción  **ESP32**


---

##  Implementaciones

### 🟢 1. Código básico

```cpp
#define LED_BUILTIN 2
#define DELAY 500

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(DELAY);
  digitalWrite(LED_BUILTIN, LOW);
  delay(DELAY);
}


### 🟢 2 Codigo con salida serie

#define LED_BUILTIN 2
#define DELAY_TIME 1000

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("ON");
  delay(DELAY_TIME);

  digitalWrite(LED_BUILTIN, LOW);
  Serial.println("OFF");
  delay(DELAY_TIME);
}

#define LED_PIN 2

void setup() {
  pinMode(LED_PIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  GPIO.out_w1ts = (1 << LED_PIN);
  Serial.println("ON");

  unsigned long startMillis = millis();
  while (millis() - startMillis < 500);

  GPIO.out_w1tc = (1 << LED_PIN);
  Serial.println("OFF");

  startMillis = millis();
  while (millis() - startMillis < 500);
}

![Foto de Marti Fontseca](https://github.com/martifontseca/PD_2025/blob/main/P1/Practica1/IMG_748E83AF-4ED5-4C90-9EE6-65B3522506C6.JPEG)


### 🟢 3. Acceso directo a registros + espera activa

#define LED_PIN 2

void setup() {
  pinMode(LED_PIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  GPIO.out_w1ts = (1 << LED_PIN);
  Serial.println("ON");

  unsigned long startMillis = millis();
  while (millis() - startMillis < 500);

  GPIO.out_w1tc = (1 << LED_PIN);
  Serial.println("OFF");

  startMillis = millis();
  while (millis() - startMillis < 500);
}


### 🟢 4. Alternativas para medir frecuencia


#define LED_PIN 2

void setup() {
  pinMode(LED_PIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  Serial.println("ON");

  while(millis() % 2 == 0);

  digitalWrite(LED_PIN, LOW);
  Serial.println("OFF");

  while(millis() % 2 == 1);
}

## 🟢 Diagrama de flujo

┌──────────────┐
│    setup()   │
│ - pinMode    │
│ - Serial.begin │
└──────┬───────┘
       │
 ┌─────▼─────┐
 │   loop()  │
 └─────┬─────┘
       │
┌──────▼──────┐
│ Encender LED│
│ Serial "ON" │
└──────┬──────┘
       │
┌──────▼──────┐
│ Esperar     │
└──────┬──────┘
       │
┌──────▼──────┐
│ Apagar LED  │
│ Serial "OFF"│
└──────┬──────┘
       │
┌──────▼──────┐
│ Esperar     │
└──────┬──────┘
       │
      (loop)



 Tiempo libre del procesador

Durante las esperas activas (delay() o while()), el procesador no ejecuta otras tareas, pero tampoco realiza trabajo útil (bloqueo activo). Esto significa que:

    Ocupado útil: solo unos pocos microsegundos por ciclo.

    Tiempo "libre" (esperando): ~99.9% del tiempo.

 Conclusiones

El acceso directo a registros ofrece mayor eficiencia y permite alcanzar frecuencias de parpadeo más elevadas.

Las funciones estándar de Arduino mejoran la legibilidad del código, pero introducen latencias y reducen la precisión temporal.

El uso de espera activa mediante millis() permite evitar el uso de delay(), manteniendo el control del flujo, aunque a costa de un mayor uso de CPU.

La comunicación por puerto serie impacta negativamente en el rendimiento, reduciendo considerablemente la frecuencia máxima de parpadeo.

El ESP32 demuestra una alta capacidad de respuesta, siempre que no se vea bloqueado por llamadas a delay() o el uso intensivo del puerto serie.
