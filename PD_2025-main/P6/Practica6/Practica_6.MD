# PRACTICA 6 :  Buses de comunicación II (SPI)  

El objetivo de la practica es comprender el funcionamiento del bus spi 

## Ejercicio Practico 1 LECTURA/ESCRITURA DE MEMORIA SD

El código utilizado es el siguiente:
```c 
#include <SPI.h>
#include <SD.h>

File myFile; // Objeto para manejar el archivo en la tarjeta SD

void setup() {
  Serial.begin(9600); // Inicialización de la comunicación serial
  
  Serial.print("Iniciando SD ...");
  
  // Inicialización de la tarjeta SD en el pin 4
  if (!SD.begin(4)) {
    Serial.println("No se pudo inicializar la tarjeta SD");
    return;
  }
  Serial.println("Inicialización exitosa de la tarjeta SD");
  
  // Intentar abrir el archivo "archivo.txt" en la tarjeta SD
  myFile = SD.open("archivo.txt");
  
  // Verificar si se pudo abrir el archivo correctamente
  if (myFile) {
    Serial.println("archivo.txt:");
    
    // Leer el contenido del archivo y enviarlo por el puerto serial
    while (myFile.available()) {
      Serial.write(myFile.read());
    }
    
    // Cerrar el archivo después de leerlo
    myFile.close();
  } else {
    // En caso de que ocurra un error al abrir el archivo
    Serial.println("Error al abrir el archivo");
  }
}

void loop() {
  // El bucle loop está vacío ya que no se requiere ninguna tarea repetitiva
}
```

### 1. Descibir la salida por el puerto serie 

Este programa ejemplifica cómo inicializar una tarjeta SD, abrir un archivo de texto almacenado en ella llamado "archivo.txt", leer su contenido y enviarlo a través del puerto serial. Al ejecutarse, el programa monta la 
tarjeta SD, accede al archivo especificado, y transmite línea por línea su contenido al monitor serie, permitiendo visualizar directamente el texto almacenado en la tarjeta. De esta forma, el usuario puede verificar 
fácilmente el contenido del fichero mediante la consola serial.

### 2. Explicar el funcionamiento 

El programa comienza incluyendo las bibliotecas <SPI.h> y <SD.h>, necesarias para manejar la comunicación SPI y el acceso a archivos en la tarjeta SD, respectivamente. Se declara un objeto File myFile para gestionar el 
archivo a leer. En la función setup(), se inicializa la comunicación serie a 9600 baudios y se intenta inicializar la tarjeta SD en el pin 4 mediante SD.begin(). Si la inicialización falla, se imprime un mensaje de error;
en caso contrario, se abre el archivo "archivo.txt" con SD.open(). Si el archivo se abre correctamente, se lee su contenido byte a byte en un bucle while y se envía a través del puerto serie para su visualización. 
Finalmente, una vez leído todo el archivo, se cierra con myFile.close(), asegurando el correcto manejo de recursos.



 
 


