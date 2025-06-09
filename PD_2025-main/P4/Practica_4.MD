# PRACTICA 4 :  SISTEMAS OPERATIVOS EN TIEMPO REAL  

El objetivo de la practica es comprender el funcionamiento de un sistema operativo en tiempo Real.

Para lo cual realizaremos una practica  donde  generaremos varias tareas  y veremos como se ejecutan dividiendo el tiempo de uso de la cpu.


## Ejercicio Practico 1 


### 1. Descibir la salida por el puerto serie 

El codigo utilizado es el siguiente:

```c
#include <Arduino.h>
#include <FreeRTOS.h>

const int led1 = 2; // Pin of the LED

// Function prototype
void anotherTask(void * parameter);

void setup()
{
  Serial.begin(112500);
  /* we create a new task here */
  xTaskCreate(
  anotherTask, /* Task function. */
  "another Task", /* name of task. */
  10000, /* Stack size of task */
  NULL, /* parameter of the task */
  1, /* priority of the task */
  NULL); /* Task handle to keep track of created task */
}
/* the forever loop() function is invoked by Arduino ESP32 loopTask */
void loop()
{
  Serial.println("this is ESP32 Task");
  delay(1000);
}
/* this function will be invoked when additionalTask was created */
void anotherTask( void * parameter )
{
  /* loop forever */
  for(;;)
  {
  Serial.println("this is another Task");
  delay(1000);
  }
  vTaskDelete(NULL);
}
```
A la salida del puerto serie tenemos este mensaje:
![Salida serie Practica 4A](Practica4A.png)

Donde podemos ver que se están ejecutando dos tareas simultáneamente en los dos núcleos diferentes del microcontrolador.

### 2. Explicar el funcionamiento 

Este código ejemplifica el uso de FreeRTOS, un sistema operativo en tiempo real que permite la ejecución concurrente de múltiples tareas en microcontroladores como el ESP32. Para utilizarlo, es necesario incluir la 
biblioteca <FreeRTOS.h>. Se define una constante led1 que representa el pin al que está conectado un LED (aunque en este ejemplo no se interactúa con él directamente). Luego, se declara una función llamada anotherTask, que
representa una tarea independiente creada con FreeRTOS; su funcionalidad consiste en imprimir un mensaje por el puerto serie cada segundo dentro de un bucle infinito. Aunque se incluye la llamada vTaskDelete(NULL) para
eliminar la tarea, esta nunca se ejecuta debido a la naturaleza infinita del bucle. En la función setup(), se inicia la comunicación serial y se crea la tarea anotherTask mediante xTaskCreate(), pasando como parámetros: la 
función a ejecutar, el nombre de la tarea ("another Task"), el tamaño del stack (10000 bytes), una prioridad (1, donde 0 es la más baja), y un puntero a un identificador de la tarea (NULL en este caso). Finalmente, la
función loop() ejecuta su propio mensaje por el monitor serie cada segundo, lo que demuestra que ambas tareas —la del loop() y la creada con FreeRTOS— se ejecutan de forma concurrente. Este ejemplo básico ilustra cómo 
FreeRTOS puede ser utilizado para estructurar programas multitarea en sistemas embebidos.







