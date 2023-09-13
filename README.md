# Medidor de Luminosidade, Temperatura e Umidade

O equipamento tem como funcionalidade medir os valores de umidade, temperatura e luminosidade em tempo real e emitir um alerta luminoso e um sonoro caso os valores se encontrem fora de faixas de tolerância.

## Materiais Utilizados
1. MCU (Atmega 328P) - Arduino Uno R3
2. LDR + Resistor 10KOhm
3. DHT-11 (Sensor de temperatura e umidade)
4. LCD 16x2
5. Bateria de 9V
6. RTC (Real Time Clock)
7. protoboard
8. jumpers
9. LED vermelho
10. Resistores
11. Suporte para bateria
12. Buzzer
13. Aplicativo Arduino

## Instalação
O primeriro passo é instalar o LCD no Arduino de acordo com a imagem abaixo, além de instalar a biblioteca LiquidCristal disponível no link: (https://www.arduinolibraries.info/libraries/liquid-crystal). Ele será responsável por exibir os valores na tela do Display.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/70ce23c5-c3da-4198-8a1a-f2b06a22d894) 
Feito isso, instale o DHT-11 de acordo com a seguinte imagem, inserindo-o no protoboard e baixe a biblioteca dht disponível neste site:(https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-sensor-de-umidade-e-temperatura-dht11). Ele é dispositivo que mede temperatura e umidade. Para que todos os dispositivos deste projeto estejam com tensão de 5 volts, basta ligar todos na mesma linha do protoboard que o LCD está conectado à entrada de 5 volts. O mesmo vale para a entrada GND. 
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/f4167e07-7cfe-4d9b-a040-96d0d4db02a1) 
Depois, instale o LDR de acordo com a imagem abaixo, que é o responsável por medir a luminosidade.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/ab3739a8-6075-4bc1-98c2-e1a67f420f36)
Agora, o próximo passo é instalar o RTC, para isso, faça como mostra a imagem, além disso, baixe a biblioteca rtc disponível nesse site: (https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-modulo-real-time-clock-rtc-ds1302). Ele servirá como relógio para marcar o tempo de 1 minuto entre as medições.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/21675191-21c1-47e3-a35c-6b455de57ba7) 
Instale também o buzzer de acordo com a imagem, que irá emitir o alerta sonoro dependendo dos resultados.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/c4251f48-950b-4d9e-8bf8-35986afac96b)
Agora instale o led vermelho de acordo com a próxima imagem. Ele será configurado para acender quando algum dos valores estiver fora dos limites estabelecidos.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/38ca274e-b3e0-48ac-ba24-6f65b0324a81)
Instale a biblioteca Adafruit Sensor disponível neste link:(https://www.arduinolibraries.info/libraries/adafruit-unified-sensor)
Para instalar as bibliotecas no aplicativo arduino, abra o aplicativo, clique em "Sketch", escolha a opção "Incluir Biblioteca" e depois  "Adicionar Biblioteca .ZIP...", selecione o arquivo ZIP da biblioteca baixada e aperte "OK". Siga este procedimento para as quatro bibliotecas: DHT, RTClib, Adafruit Sensor e LiquidCrystal.
Agora, o próximo passo é abrir aplicativo do arduíno no computador e utilizar o seguinte código:

```
#include <EEPROM.h>

#include <RTClib.h>

#include <LiquidCrystal.h>

#include <dht.h>

#include <Adafruit_Sensor.h>


RTC_DS3231 rtc;
char daysOfTheWeek[7][12] = {"Domingo", "Segunda", "Terça", "Quarta", "Quinta", "Sexta", "Sábado"};
const int ldrPin = A0;
int Pinbuzzer = 10; //Porta do buzzer

//Variaveis preenchidas a cada leitura
int contTemp = 0;
int contUmid = 0;
int contLumi = 0;

//Variaveis para medias geradas de 1 em 1 minuto
int medTemp = 0;
int medUmid = 0;
int medLumi = 0;
int cont = 0;


#define COMMON_ANODE

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

dht DHT;
uint32_t timer = 0;
unsigned long time;

void setup() {
  pinMode(9, OUTPUT);
  pinMode(pinoR, OUTPUT);
  pinMode(pinoG, OUTPUT);
  pinMode(pinoB, OUTPUT);
  Serial.begin(9600);
  lcd.begin(16, 2);
  lcd.clear();
  pinMode(Pinbuzzer, OUTPUT);
  
  if (!rtc.begin()) {
    Serial.println("DS3231 não encontrado"); // Verificação rtc
    while (1);
  }
  
  if (rtc.lostPower()) {
    Serial.println("DS3231 OK!");
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }

  delay(100);
}


void loop() {
  DateTime now = rtc.now();
  
  if (now.second() % 5 == 0) { //captador de valores atuais para poder realizar as medias
    DHT.read11(A1);
    int valorLdr = analogRead(ldrPin);
    int brilho = map(valorLdr, 0, 1023, 0, 100);
    
    Serial.print("Luminosidade = ");
    Serial.print(brilho);
    Serial.println("%");
 
    Serial.print("Umidade = ");
    Serial.print(DHT.humidity);
    Serial.print("%");
 
    Serial.print("Temperatura = ");
    Serial.print(DHT.temperature); 
    Serial.println(" Celsius  ");

    int temp = DHT.temperature;
    int umid = DHT.humidity;

    contTemp += temp;
    contUmid += umid;
    contLumi += brilho;

    cont++;
  }

  if (now.second() == 58) { //calcula a media de um minuto
    medTemp = contTemp / cont;
    medUmid = contUmid / cont;
    medLumi = contLumi / cont;
   
    if (medTemp < 15 || medTemp > 25 || medLumi > 30 || medUmid < 30 || medUmid > 50) {
//      saveDataToEEPROM(medTemp, medUmid, medLumi, now);

        String log_eeprom = "";
        log_eeprom += String(medUmid) + "%|";
        log_eeprom += String(medTemp) + "*C|";
        log_eeprom += String(medLumi) + "%";
        log_eeprom += "-" + String(now.day(), DEC) +"/"+ String(now.month(), DEC) +"/"+ String(now.year(), DEC) + ',';  // String para salvar os valores na EEPROM

        //salvar string na eeprom
        for (unsigned long i = 0; i < 1024; ++i)
        {
            EEPROM.write(i, log_eeprom[i]);
        }            
      digitalWrite(Pinbuzzer, HIGH);
      digitalWrite(9, HIGH);
    }
  }
      
  lcd.setCursor(0, 1);
  lcd.print("Temp: ");
  lcd.print(medTemp);
  lcd.write(B11011111);
  lcd.print("C");
  
  lcd.setCursor(8, 0);
  lcd.print("Lum:");
  lcd.print(medLumi);
  lcd.print("%");
  
  lcd.setCursor(0, 0);
  lcd.print("Umd:");
  lcd.print(medUmid);
  lcd.print("%");

  Serial.print("Data: ");
  Serial.print(now.day(), DEC);
  Serial.print('/');
  Serial.print(now.month(), DEC);
  Serial.print('/');
  Serial.print(now.year(), DEC);
  Serial.print(" / Dia: ");
  Serial.print(daysOfTheWeek[now.dayOfTheWeek()]);
  Serial.print(" / Horas: ");
  Serial.print(now.hour(), DEC);
  Serial.print(':');
  Serial.print(now.minute(), DEC);
  Serial.print(':');
  Serial.print(now.second(), DEC);
  Serial.println();
  
  //Mostra log_eeprom no serial
   String log_serial = "";     
   for(int i = 0; i < EEPROM.length(); ++i)
   {
        log_serial += char(EEPROM.read(i));
   }      
   Serial.print(log_serial);
  delay(1000);
}

void saveDataToEEPROM(int temp, int umid, int lumi, DateTime timestamp) {
  int address = 0;
  int dataSize = sizeof(int) * 4;

  EEPROM.put(address, temp);
  address += sizeof(int);
  EEPROM.put(address, umid);
  address += sizeof(int);
  EEPROM.put(address, lumi);
  address += sizeof(int);
  EEPROM.put(address, timestamp.unixtime());
  Serial.println("Dados fora do padrão salvos na EEPROM.");
}

```
Caso as entradas utilizadas no arduíno não coincidam com os valores de entrada do código, basta alterar os valores de entrada do código para que eles se adaptem ao seu projeto.

## Uso 
Para o equipameto funcionar, conecte o arduino ao computador, verifique e carregue o código e inicie a compilação, ele calculará as médias dos valores de temperatura, luminosidade e umidade obtidos ao longo de um minuto de medições, realizadas a cada 2 segundos, e as exibirá no display. Caso a temperatura não esteja entre 15 e 25 graus Celsius, ou a luminosidade for maior que 30%, ou a umidade não estiver entre 30 e 50%, o alarme sonoro será ativado através do buzzer e o led vermelho ficará aceso. Para o equipamento funcionar sem estar conectado ao computador, conecte a bateria no arduino depois de já ter carregado o código.

## Créditos

(https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-sensor-de-umidade-e-temperatura-dht11)
(https://victorvision.com.br/blog/display-lcd-16x2/)
(https://mundoprojetado.com.br/ldr-como-usar/)
(https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-modulo-real-time-clock-rtc-ds1302)
(https://mundoprojetado.com.br/buzzer-como-usar-com-o-arduino/#:~:text=O%20circuito%20do%20buzzer%20%C3%A9,um%20pino%20digital%20do%20Arduino.)
(https://www.arduinolibraries.info/libraries/liquid-crystal)
(https://tecdicas.com/como-acender-e-piscar-um-led-no-arduino/)
(https://www.arduinolibraries.info/libraries/adafruit-unified-sensor)
