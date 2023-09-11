# Medidor de Luminosidade, Temperatura e Umidade

Descrição
O equipamento tem como funcionalidade medir os valores de umidade, temperatura e luminosidade em tempo real e emitir um alerta caso os valores se encontrem fora de faixas de tolerância.

Materiais Utilizados
1 MCU (Atmega 328P) - Arduino Uno R3
1 LDR + Resistor 10KOhm
1 DHT-11 (Sensor de temperatura e umidade)
1 LCD 16x2
1 Bateria de 9V
1 RTC (Real Time Clock)
1 protoboard
jumpers
LEDs
Resistores
1 Suporte para bateria

Instalação
O primeriro passo é instalar o DHT-11 no Arduino de acordo com a imagem abaixo:![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/a77c4a26-1476-445a-835e-d377f9e8e738) e inserir o DHT-11 no protoboard. Instale a biblioteca dht disponível neste site: https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-sensor-de-umidade-e-temperatura-dht11. Feito isso, digite os seguintes comandos no aplicativo Arduino para incluir a biblioteca dht:
#include "dht.h"
const int pinoDHT11 = A2;
dht DHT;
eles devem estar antes dos métodos void setup() e void loop()  








## Créditos

https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-sensor-de-umidade-e-temperatura-dht11
