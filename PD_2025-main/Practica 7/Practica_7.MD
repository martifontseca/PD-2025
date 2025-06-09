# PRACTICA 7 :  Buses de comunicación III (I2S)  

El objetivo de la practica actual es describir el funcionamiento del bus I2S y realizar una practica para comprender su  funcionamiento 

## Ejercicio Practico 1  Reproducción desde memoria interna

### 1. Descibir la salida por el puerto serie 

El código utilizado es el siguiente:
```c
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"

AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;

void setup()
{
    Serial.begin(115200);
    in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
    aac = new AudioGeneratorAAC();
    out = new AudioOutputI2S();
    out -> SetGain(0.125);
    out -> SetPinout(26,25,22);
    aac->begin(in, out);
}
void loop()
{
    if (aac->isRunning()) 
    {
        aac->loop();
    } 
    else 
    {
        aac -> stop();
        Serial.printf("Sound Generator\n");
        delay(1000);
    }
}
```

Este código ejemplifica cómo reproducir un archivo de audio codificado en AAC (Advanced Audio Coding) utilizando un microcontrolador junto con un DAC MAX98357 conectado vía protocolo I2S para convertir la señal digital en 
una señal analógica que se puede reproducir en altavoces. La salida por el puerto serie muestra datos relacionados con la señal analógica que se está procesando para la reproducción. En este ejemplo, el audio reproducido 
corresponde a un archivo llamado sampleaac, que contiene un fragmento famoso de Homer Simpson hablando sobre la termodinámica ("¡En esta casa obedecemos las leyes de la TERMODINÁMICA!"), permitiendo comprobar el correcto 
funcionamiento del sistema de decodificación y reproducción.



### 2. Explicar el funcionamiento 
El código utiliza las bibliotecas AudioGeneratorAAC.h, AudioOutputI2S.h y AudioFileSourcePROGMEM.h para reproducir un archivo de audio AAC almacenado en la memoria del programa (PROGMEM) en un microcontrolador, usando un 
DAC MAX98357 conectado vía protocolo I2S. Se crean objetos para manejar la fuente del audio (AudioFileSourcePROGMEM), el generador y decodificador AAC (AudioGeneratorAAC), y la salida I2S (AudioOutputI2S). En la función
setup(), se inicializa la comunicación serial a 115200 baudios, se instancia el archivo sampleaac en PROGMEM, se configura el generador AAC y la salida I2S, ajustando la ganancia y los pines del protocolo I2S (pines 26, 25
y 22). En el loop(), se comprueba si la reproducción está activa con aac->isRunning(), procesando la reproducción con aac->loop() mientras esté en ejecución; si no, se detiene el generador con aac->stop(), imprime un 
mensaje y espera un segundo antes de repetir el ciclo. De esta manera, el sistema decodifica y reproduce el audio digital, enviando la señal analógica a los altavoces a través del DAC, permitiendo escuchar el contenido 
almacenado, en este caso el fragmento sampleaac con la voz de Homer Simpson.
