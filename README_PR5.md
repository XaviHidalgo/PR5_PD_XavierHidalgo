# PR5_PD_XavierHidalgo

## PRACTICA 5:  Buses de comunicación I (introducción y I2c)

### Ejercicio Practico 1 ESCÁNER I2C

**PROGRAMA:**

``` cpp
#include <Arduino.h>
#include <Wire.h>

void setup()
{
Wire.begin();
Serial.begin(115200);
while (!Serial){
// Leonardo: wait for serial monitor
Serial.println("\nI2C Scanner");
}
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
nDvices++;
Serial.print("I2C device found at address 0x");
if (address<16) {
Serial.print("0");
Serial.print(address,HEX);
Serial.println(" !");
}
}
}
}
```
**INFORME:**

El anterior programa es un escáner o programa de reconocimiento de periféricos conectados a un bus I2C. 

Para saber la cantidad de periféricos connectadosa nuestro bus (objetivo de un escáner), enviaremos a las 128 direcciones posibles (ya que el bus I2C guarda las direcciones de los dispositivos esclavos en 7 bits) una señal buscando otra señal de respuesta. Si es así, sabremos que hay un dispositivo connectado al bus y además su dirección para comunicarnos con él. Debido a las resistencias de pull-up que incluye el bus I2C, el circuito se mantiene en alta impedancia continuamente así que, si acertamos la dirección de un periférico, lo sabremos ya que como respuesta el periférico variará esta alta impedancia. Esta variación, en el programa la entenderemos como un '0'.

A continuación explico el funcionamiento del programa y la salida que tenemos por el pueto série tras su ejecucción:

Si nos fijamos en el inicio del código, vemos que, como casi siempre hacemos incluimos la libreria Arduino.h (ya que estamos programando con un esquema tipo Arduino y es posible que nos evite problemas) y, como nueva incorporación, incluimos Wire.h. Esta última libreria nos permitira comunicarnos sobre el bus I2C, permitiendonos iniciar y acabar transmisiones en distintas direcciones.

Más adelante, en el void_setup(), inicializamos el bus série (como siempre) y el bus I2C a partir de la instancia "Wire.begin;". Después nos encontramos con un while: "while (!Serial)". Como vemos, se ejecutará cuando el bus série este listo, tras eso escribe por él "\nI2C Scanner", diciéndonos que el bus série esta operativo y que se inicia el escáner.

EL escáner se inicia en el void_loop(). Como he escrito al principio, necesitamos enviar por el bus todas las direcciones de dispositivo y ver si conseguimos respuesta de algún periférico tras ello. Esto lo haremos con un bucle for. Para que funcione el bucle necesitamos declarar una serie de variables, declararemos: nDevices (tipo int, define la cantidad de periféricos connectados), error (tipo byte, nos indicará si ha encontrado en x dirección un dispositivo) y address (tipo byte, aqui guardaremos las direcciones). El bucle recorrerá acabará cuando haya comprobado todas las direcciones, es decir, empieza comprobando la dirección 00000000 y va sumando +1 en binario hasta llegar a la última, la **0**1111111 (el primer 0 se debe a que el tipo de variable byte son lógicamente 8 bits y las direcciones, como he escrito antes, son de 7).

Bien, enviaremos cada dirección por el bus I2C gracias a la función .beginTransmission() de Wire.h de la siguiente forma:
``` cpp
Wire.beginTransmission(address);
```
Tras iniciarla, la cerraremos con la función .endTransmission() de la misma libreria la cual nos devuelve un valor tipo byte que lo guardaremos en nuestra variable error:
``` cpp
error = Wire.endTransmission();
```
error valdrá 0 si la conexión ha sido satisfactoria, como he comentado en el segundo párrafo, la variación de alta impedancia y, por lo tanto una respuesta por parte del periférico, la entendemos como un 0. Entonces, tras tener un valor en error, con un if() preguntamos si error vale 0 para, si es así, enviar por el puerto série la dirección del dispositivo encontrado en **hexadecimal**:
``` cpp
if (error == 0)
{
nDvices++;
Serial.print("I2C device found at address 0x");
if (address<16) {
Serial.print("0");
Serial.print(address,HEX);
Serial.println(" !");
}
}
```

**CIRCUITO EN PRÁCTICA:**

Periféricos connectados : 2

Conexiones : cada periférico connectado a: PIN 21(representa el SDA del bus), PIN 22(representa el SCL del bus), GND, Vcc (3.3 V)

Imagen del montage:

![Im1](https://github.com/XaviHidalgo/PR5_PD_XavierHidalgo/blob/main/20240318_194215.jpg)


### Ejercicio Practico 2

Este ejercicio nos pedía utilizar dos dispositivos, connectarlos al bus I2C y hacerlos funcionar. En mi caso he utilizado un sensor de temperatura y humedad y un display OLED.

Después connectar los dos periféricos al bus, utilicé el programa anterior para comprobar que se establecia comunicación con el MP correctamente. Tras eso y ver que los dos estan correctamente connectados, envié el siguiente programa:


