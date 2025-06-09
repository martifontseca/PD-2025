# PRACTICA 2 : INTERRUPCIONES 

El objetivo de esta práctica es comprender el funcionamiento de las interrupciones en sistemas embebidos.

Para ello, se implementará un control periódico de dos LEDs y una entrada digital. La activación de dicha entrada modificará la frecuencia de oscilación de uno de los LEDs, mientras que el otro continuará con su frecuencia original. De esta manera, se podrá observar el efecto de las interrupciones y su aplicación en el control selectivo de tareas.


## Introducción teórica  

¿Qué es una interrupción hardware?

A un nivel básico, una interrupción es una señal que interrumpe la actividad normal de nuestro microprocesador y salta a atenderla. Hay tres eventos que pueden disparar una interrupción:

 
* Un evento hardware, previamente definido.
* Un evento programado, o Timer
* Una llamada por software.
 
Cuando ocurre un evento que genera una interrupción, la ejecución normal del microcontrolador se suspende de forma ordenada para poder reanudarse más adelante, y el control del programa salta a una función especial denominada Interrupt Service Handler (ISH), o rutina de servicio de interrupción.

Una vez que el ISH finaliza su ejecución, el procesador regresa al punto exacto donde se detuvo, continuando su flujo de instrucciones como si nada hubiera ocurrido.

El concepto de interrupción surge de la necesidad de reaccionar de manera inmediata ante ciertos eventos electrónicos que ocurren con una rapidez tal que no admiten demora.
Esto puede deberse a la urgencia del evento o al riesgo de perder información irrecuperable si no se responde con suficiente rapidez.

¿Por qué usar interrupciones?
Puede surgir la duda:
¿Qué tipo de evento es tan urgente que no puede esperar? ¿Acaso un procesador no es lo suficientemente rápido como para verificar constantemente si ocurre algo?

La respuesta es: depende. Aunque los procesadores modernos son veloces, siguen ejecutando tareas secuencialmente. Si se recurre a una técnica como el polling —consistente en 
revisar periódicamente el estado de una entrada o registro— es posible que una señal breve desaparezca antes de ser detectada. En esos casos, ni siquiera se sabrá que ocurrió. Además, técnicas como el uso de delay() pueden bloquear el flujo del programa, dificultando la detección oportuna de eventos.

Ventajas de las interrupciones
Las interrupciones ofrecen una forma elegante y eficiente de gestionar eventos externos o internos. Basta con definir una función específica que se ejecutará automáticamente 
cuando ocurra una determinada interrupción, sin necesidad de comprobar constantemente el estado de una señal. El programa principal puede continuar con su lógica habitual, y solo se interrumpirá si realmente ocurre el evento configurado.

De hecho, el comportamiento humano funciona de forma similar: muchas veces respondemos a estímulos inesperados que interrumpen nuestras rutinas diarias, como cuando suena una alarma mientras estamos viendo nuestra serie favorita.
    struct Button {
      const uint8_t PIN;
      uint32_t numberKeyPresses;
      bool pressed;
    };

    Button button1 = {18, 0, false};

    void IRAM_ATTR isr() {
      button1.numberKeyPresses += 1;
      button1.pressed = true;
    }

    void setup() {
      Serial.begin(115200);
      pinMode(button1.PIN, INPUT_PULLUP);
      attachInterrupt(button1.PIN, isr, FALLING);
    }

    void loop() {
      if (button1.pressed) {
          Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
          button1.pressed = false;
      }

      //Detach Interrupt after 1 Minute
      static uint32_t lastMillis = 0;
      if (millis() - lastMillis > 60000) {
        lastMillis = millis();
        detachInterrupt(button1.PIN);
         Serial.println("Interrupt Detached!");
      }
    }

### Informe de la practica
Durante esta práctica se exploró el uso de interrupciones en un microcontrolador ESP32, enfocándonos en cómo estas permiten responder de forma inmediata a eventos externos, sin necesidad de emplear técnicas de sondeo o polling. El objetivo fue ilustrar cómo, mediante interrupciones, podemos mantener un control eficiente del sistema sin desperdiciar ciclos de CPU en comprobaciones constantes.

Analogía Explicativa
Para entender mejor el funcionamiento de las interrupciones, consideremos una situación cotidiana: estás viendo tu serie favorita en la televisión, mientras esperas a que llegue alguien (un amigo, tu pareja, etc.).

Tienes dos opciones para saber si ha llegado:

Polling (sondeo): Te levantas cada dos minutos a ver si está esperando en la puerta. Si llegó justo después de la última vez que miraste y se fue antes de la siguiente, nunca te enteraste. Esto consume tu atención y rompe tu concentración en la serie.

Interrupción: Instalas un timbre. Cuando tu visitante llega, pulsa el timbre. En ese momento, interrumpes tu serie, pausas la imagen, dejas el refresco, y vas a abrir. Luego, vuelves al sofá, reanudas el capítulo y sigues disfrutando.
Esto es precisamente lo que hacen las interrupciones: solo reaccionan cuando realmente pasa algo, y te permiten seguir con tu rutina mientras tanto.

Del mismo modo, el microcontrolador puede ejecutar su programa principal sin preocuparse por consultar constantemente si una señal ha cambiado. Si configuramos correctamente una interrupción, esta se activará automáticamente cuando ocurra un evento específico (por ejemplo, un cambio de estado en una entrada digital), ejecutará la rutina asociada y luego continuará con el flujo normal del programa.

## Tipos de interrupciones 

De los tres  sucesos que pueden disparar una interrupción

 
* Un evento hardware,
* Un evento programado, o Timer
* Una llamada por software.
 
Nos encontramos con que Arduino no soporta las interrupciones por software y punto.

¿Y entonces porque hemos hablado de ellas? Pues, porque otros entornos de programación las aceptan y no será raro que en el futuro Arduino también.

Los eventos programados o Timers, y hoy los utilizaremos en la practica , también veremos hoy interrupciones disparadas por hardware.


## Interrupciones en ESP32

En el ESP32, podemos definir una función de rutina de servicio de interrupción que se llamará cuando un pin GPIO cambie el valor de su señal.

Con una placa ESP32, todos los pines GPIO pueden ser configurados para funcionar como entradas de solicitud de interrupción.

### Adjuntar la interrupción a un PIN GPIO

En el IDE de Arduino, usamos una función llamada attachInterrupt() para establecer una interrupción en base a un pin por pin. La sintaxis recomendada es la siguiente.

    attachInterrupt(GPIOPin, ISR, Mode);

Esta función toma tres parámetros:

* GPIOPin: Establece la clavija GPIO como una clavija de interrupción, que le dice al ESP32 qué clavija debe monitorear.
* ISR: Es el nombre de la función que se llamará cada vez que se dispare la interrupción.
* Mode: Define cuándo se debe disparar la interrupción. Cinco constantes están predefinidas como valores válidos:
    * LOW	Los disparadores interrumpen cuando el pin está LOW
    * HIGH	Los disparadores interrumpen cuando el pin es HIGH
    * CHANGE	Los disparadores interrumpen cuando el pin cambia de valor, de HIGH a LOW o LOW a HIGH
    * FALLING	Los disparadores interrumpen cuando el pin va de HIGH a LOW
    * RISING	Los disparadores interrumpen cuando el pin va de LOW a HIGH


### Desconectar la interrupcion de un GPIO 

Opcionalmente, puedes llamar a la función de separación de interrupciones cuando ya no quieras que el ESP32 monitorice un pin. La sintaxis a utilizar es la siguiente.

     detachInterrupt(GPIOPin);

### Rutina de servicio de interrupcion

Esta es la función que se llama cuando  ocurre el evento dela interrupción , dede de ser corta de tiempo de ejecución . Su sintaxis es como sigue.

     void IRAM_ATTR ISR() {
          Statements;
      }


## Temporizadores en ESP32

Un temporizador es un contador interno ; que al terminar un determinado numero de cuentas  produce una interrupción 

El ESP32 tiene dos grupos de temporizadores, cada uno con dos temporizadores de hardware de propósito general. Todos los temporizadores se basan en contadores de 64 bits y preescaladores de 16 bits.

El preescalador se utiliza para dividir la frecuencia de la señal de base (normalmente 80 MHz), que luego se utiliza para incrementar/disminuir el contador de tiempo. Dado que el prescalificador tiene 16 bits, puede dividir la frecuencia de la señal del reloj por un factor de 2 a 65536, lo que da mucha libertad de configuración.

Los contadores de tiempo pueden ser configurados para contar hacia arriba o hacia abajo y apoyar la recarga automática y la recarga de software. También pueden generar alarmas cuando alcanzan un valor específico, definido por el software [2]. El valor del contador puede ser leído por el programa de software.

para entender el funcionamiento de la practica (temporizador) se debe leer el siguiente tutorial 

https://techtutorialsx.com/2017/10/07/esp32-arduino-timer-interrupts/

## Practica A interrupción por GPIO 


#include <Arduino.h>


struct Button {
const uint8_t PIN;
uint32_t numberKeyPresses;
bool pressed;
};
Button button1 = {18, 0, false};
void IRAM_ATTR isr() {
button1.numberKeyPresses += 1;
button1.pressed = true;
}
void setup() {
Serial.begin(115200);
pinMode(button1.PIN, INPUT_PULLUP);
attachInterrupt(button1.PIN, isr, FALLING);
}
void loop() {
if (button1.pressed) {
Serial.printf("Button 1 has been pressed %u times\n",
button1.numberKeyPresses);
button1.pressed = false;
}
//Detach Interrupt after 1 Minute
static uint32_t lastMillis = 0;
if (millis() - lastMillis > 60000) {
lastMillis = millis();
detachInterrupt(button1.PIN);
Serial.println("Interrupt Detached!");
}
}

Este código configura un botón conectado al pin 18 del ESP32 y utiliza una interrupción para contar cuántas veces se ha presionado el botón.
Se define una estructura Button que contiene el pin del botón, el número de presiones y el estado del botón (pressed).
La función isr() se ejecuta cuando el botón se presiona, incrementando el contador numberKeyPresses y marcando que el botón fue presionado (pressed = true).
En el setup(), se configura el pin 18 como entrada con resistencia pull-up y se adjunta una interrupción al pin para detectar la caída de señal (cuando el botón se presiona).
En el loop(), si el botón ha sido presionado, el código imprime cuántas veces se ha presionado el botón en el Monitor Serie. Luego, la variable pressed se restablece a false.
Después de 1 minuto (60000 milisegundos), se desactiva la interrupción, y se imprime un mensaje indicando que la interrupción ha sido detenida.
El archivo de configuración en platformio.ini establece la plataforma y la velocidad del monitor serie.
El contador se desfasa debido a los rebotes que suceden
Hay veces que no cuenta todos los rebotes en el flanco. 
En resumen, el programa cuenta las veces que se presiona un botón, lo muestra por el Monitor Serie y desactiva la interrupción después de un minuto.




## Practica B interrupción por timer 

El codigo de la practica es el siguiente 
```
#include <Arduino.h>


volatile int interruptCounter = 0;  // Se declara como volatile
int totalInterruptCounter = 0;      // No necesita ser volatile


hw_timer_t *timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;


void IRAM_ATTR onTimer() {
 portENTER_CRITICAL_ISR(&timerMux);  // Entramos en el modo crítico en la ISR
 interruptCounter++;  // Incrementamos el contador de interrupciones
 portEXIT_CRITICAL_ISR(&timerMux);  // Salimos del modo crítico en la ISR
}


void setup() {
 Serial.begin(115200);


 // Inicialización del temporizador (1us = 80)
 timer = timerBegin(0, 80, true);  // Timer 0, prescaler 80 (para 1us)
 timerAttachInterrupt(timer, &onTimer, true);  // Adjuntamos la interrupción
 timerAlarmWrite(timer, 1000000, true);  // Alarma en 1 segundo (1000000 us)
 timerAlarmEnable(timer);  // Habilitamos la alarma
}


void loop() {
 if (interruptCounter > 0) {  // Si ha ocurrido una interrupción
   portENTER_CRITICAL(&timerMux);  // Entramos en el modo crítico
   interruptCounter--;  // Decrementamos el contador de interrupciones
   portEXIT_CRITICAL(&timerMux);  // Salimos del modo crítico


   totalInterruptCounter++;  // Contamos la interrupción total
   Serial.print("An interrupt has occurred. Total number: ");
   Serial.println(totalInterruptCounter);  // Imprimimos el total
 }
}


Este código configura un temporizador hardware en el ESP32 para generar una interrupción cada 1 segundo, utilizando una rutina de servicio de interrupción (ISR) llamada onTimer(), que incrementa un contador 
(interruptCounter) cada vez que se activa el temporizador. En la función setup(), se inicializa el temporizador con un prescaler de 80, lo que hace que cuente en microsegundos, y se configura para que dispare una 
interrupción cada 1.000.000 de microsegundos (1 segundo). Además, se habilita la interrupción y se asocia la ISR correspondiente. En la función loop(), se comprueba si interruptCounter ha sido incrementado por la ISR y, 
si es así, se accede de forma segura a las variables compartidas usando una sección crítica (portENTER_CRITICAL y portEXIT_CRITICAL) para evitar condiciones de carrera. En ese bloque, se decrementa interruptCounter y se 
incrementa totalInterruptCounter, que representa el número total de interrupciones procesadas. Finalmente, este valor se imprime en el monitor serie, mostrando la cuenta acumulada de interrupciones. De este modo, se 
demuestra cómo utilizar temporizadores e interrupciones para realizar tareas periódicas con precisión y eficiencia, sin recurrir a funciones bloqueantes como delay(), y asegurando un manejo correcto de la concurrencia 
entre el código principal y las ISR.


## Ejercicio Extra 


#include <Arduino.h>


const int ledPin = 2;      // Pin donde se conecta el LED
const int buttonPin1 = 4;  // Pin del primer pulsador
const int buttonPin2 = 5;  // Pin del segundo pulsador


volatile int blinkDelay = 1000;  // Frecuencia inicial de parpadeo (en milisegundos)
volatile bool ledState = LOW;    // Estado del LED
volatile unsigned long lastDebounceTime1 = 0;  // Tiempo para debouncing del pulsador 1
volatile unsigned long lastDebounceTime2 = 0;  // Tiempo para debouncing del pulsador 2
unsigned long debounceDelay = 50;  // Tiempo de espera para evitar rebotes (en milisegundos)


hw_timer_t *timer = NULL;         // Temporizador
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;


void IRAM_ATTR onTimer() {
 // Cambiar el estado del LED solo cuando el tiempo de parpadeo haya pasado
 portENTER_CRITICAL_ISR(&timerMux);
 static unsigned long previousMillis = 0; // Variable para llevar el control del tiempo
 unsigned long currentMillis = millis();
  // Solo alternamos el LED si ha pasado el tiempo configurado en blinkDelay
 if (currentMillis - previousMillis >= blinkDelay) {
   ledState = !ledState;  // Alternar el estado del LED
   digitalWrite(ledPin, ledState);  // Aplicamos el cambio de estado al LED
   previousMillis = currentMillis;  // Actualizamos el tiempo del último cambio
 }
 portEXIT_CRITICAL_ISR(&timerMux);
}


void IRAM_ATTR buttonInterrupt1() {
 if (millis() - lastDebounceTime1 > debounceDelay) {
   portENTER_CRITICAL_ISR(&timerMux);
   blinkDelay = max(100, blinkDelay - 100);  // Aumenta la frecuencia (reduce el delay)
   portEXIT_CRITICAL_ISR(&timerMux);
   lastDebounceTime1 = millis();
 }
}


void IRAM_ATTR buttonInterrupt2() {
 if (millis() - lastDebounceTime2 > debounceDelay) {
   portENTER_CRITICAL_ISR(&timerMux);
   blinkDelay += 100;  // Disminuye la frecuencia (aumenta el delay)
   portEXIT_CRITICAL_ISR(&timerMux);
   lastDebounceTime2 = millis();
 }
}


void setup() {
 Serial.begin(115200);


 pinMode(ledPin, OUTPUT);
 pinMode(buttonPin1, INPUT_PULLUP);
 pinMode(buttonPin2, INPUT_PULLUP);


 // Configurar interrupciones para los pulsadores
 attachInterrupt(buttonPin1, buttonInterrupt1, FALLING);
 attachInterrupt(buttonPin2, buttonInterrupt2, FALLING);


 // Configuración del temporizador para generar interrupciones a intervalos
 timer = timerBegin(0, 80, true);  // Timer 0, prescaler 80 para generar interrupciones a 1ms
 timerAttachInterrupt(timer, &onTimer, true);  // Asociamos la interrupción
 timerAlarmWrite(timer, blinkDelay, true);  // Configuramos el temporizador con el delay inicial
 timerAlarmEnable(timer);  // Habilitamos la alarma
}


void loop() {
 // El código en el loop es muy simple ya que todo está manejado por las interrupciones
}


Informe Breve – Ejercicio Extra: Control de Parpadeo con Pulsadores e Interrupciones

Objetivo

Este sistema permite controlar la frecuencia de parpadeo de un LED en un ESP32 utilizando dos pulsadores conectados a interrupciones por hardware: uno para aumentar y otro para disminuir la frecuencia, todo sin utilizar 
el bucle loop(). El parpadeo del LED es gestionado mediante un temporizador hardware cuya frecuencia se ajusta en tiempo real según la interacción del usuario, incrementando o reduciendo el tiempo de parpadeo. Las 
interrupciones asociadas a los pulsadores permiten una respuesta inmediata, y se implementa un pequeño retardo (debounce) para evitar rebotes mecánicos. El acceso a variables compartidas entre interrupciones y lógica de
temporización se protege mediante secciones críticas, garantizando estabilidad y sincronización. Esta implementación ofrece alta eficiencia al liberar al procesador de tareas de sondeo o espera activa, permite un control
preciso de la frecuencia y resulta fácilmente escalable para otros sistemas que requieran control de eventos temporales. Es especialmente útil como base para temporizadores, controladores PWM, señales visuales 
programables o como ejercicio práctico avanzado en el manejo de interrupciones en sistemas embebidos.





