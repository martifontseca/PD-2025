# PRACTICA 5 :  Buses de comunicación I (Introducción y I2C)  

El objetivo de la practica es comprender el funcionamiento de los buses
  sistemas de comunicación entre periféricos;  estos elementos pueden 
  ser internos o externos  al procesador.

Esta es la primera practica de  4 donde se verán los buses i2c, spi,
 i2s , usart.

Para lo cual realizaremos una practica en cada caso donde controlaremos 
 un periférico de ejemplo

## Ejercicio Practico 1  ESCÁNER I2C

Programar el siguiente codigo,  colocar varios dispositivos  I2C 

```
En el caso de ESP32 por defecto:

| I2C Device | ESP32                    |
| ---------- | ------------------------ |
| SDA        | SDA (default is GPIO 21) |
| SCL        | SCL (default is GPIO 22) |
| GND        | GND                      |
| VCC        | usually 3.3V or 5V       |
```

como en la figura siguiente 

![](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2019/09/I2C-communication-protocol-ESP32.png?resize=1024%2C577&quality=100&strip=all&ssl=1)

Las resistencias de pull-up  dependen del numero de dispositivos ; 
aunque un valor usual es  3k3 


### 1. Descibir la salida por el puerto serie 

El código utilizado es el siguiente:
```c 
#include <Arduino.h>
#include <Wire.h>

void setup()
{
    Wire.begin();
    Serial.begin(115200);
    while (!Serial);
    // Leonardo: wait for serial monitor
    Serial.println("\nI2C Scanner");
}

void loop()
{
    byte error, address;
    int nDevices;
    Serial.println("Scanning...");
    nDevices = 0;
    for(address = 1; address < 127; address++ )
    {
        // The i2c_scanner uses the return value of
        // the Write.endTransmisstion to see if
        // a device did acknowledge to the address.
        Wire.beginTransmission(address);
        error = Wire.endTransmission();
        if (error == 0)
        {
            Serial.print("I2C device found at address 0x");
            if (address<16)
            {
                Serial.print("0");
            }
            Serial.print(address,HEX);
            Serial.println(" !");

            nDevices++;
        }
        else if (error==4)
        {
            Serial.print("Unknown error at address 0x");
            if (address<16)
            {
                Serial.print("0");
            }
            Serial.println(address,HEX);
        }
    }
    if (nDevices == 0)
    {
        Serial.println("No I2C devices found\n");
    }
    else
    {
        Serial.println("done\n");
        delay(5000);
        // wait 5 seconds for next scan
    }
}
```

En este caso, la salida por el puerto serie nos indica la dirección del periférico conectado al microcontrolador mediante el protocolo I2C, expresada en formato hexadecimal. Esta información es especialmente útil durante 
la fase de desarrollo y prueba, ya que permite verificar que el dispositivo I2C está correctamente conectado y responde en la dirección esperada. El escaneo I2C identifica todos los dispositivos presentes en el bus, lo 
cual facilita la detección de errores de conexión, conflictos de dirección o fallos en la comunicación.

### 2. Explicar el funcionamiento 
Para que este código funcione correctamente con el protocolo I2C, es necesario incluir la librería <Wire.h>. En la función setup(), se inicia la comunicación I2C con Wire.begin() y la comunicación serie con 
Serial.begin(115200), estableciendo una velocidad de 115200 baudios; además, se espera a que la conexión serie esté disponible antes de imprimir un mensaje que indica el inicio del escaneo de dispositivos I2C. En la 
función loop(), se muestra un mensaje indicando que la comunicación está activa, se inicializa un contador nDevices para contabilizar los dispositivos encontrados, y se recorre un bucle desde la dirección 1 hasta la 126,
donde en cada iteración se intenta iniciar una transmisión con Wire.beginTransmission(address) y se termina con Wire.endTransmission(), que devuelve un código de error para determinar si un dispositivo respondió en esa
dirección. Si error == 0, se imprime la dirección del dispositivo encontrado en formato hexadecimal; si error == 4, se señala un error de comunicación. Al finalizar el escaneo, si se detectaron dispositivos, se imprime 
"done", y en caso contrario se espera 5 segundos antes de repetir el proceso, permitiendo así una detección continua y fiable de dispositivos I2C en el bus.
