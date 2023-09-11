# Medidor de Luminosidade, Temperatura e Umidade

O equipamento tem como funcionalidade medir os valores de umidade, temperatura e luminosidade em tempo real e emitir um alerta caso os valores se encontrem fora de faixas de tolerância.

## Materiais Utilizados
1. MCU (Atmega 328P) - Arduino Uno R3
2. LDR + Resistor 10KOhm
3. DHT-11 (Sensor de temperatura e umidade)
4. LCD 16x2
5. Bateria de 9V
6. RTC (Real Time Clock)
7. protoboard
8. jumpers
9. LEDs
10. Resistores
11. Suporte para bateria
12. Buzzer
13. Aplicativo Arduino

## Instalação
O primeriro passo é instalar o LCD no Arduino de acordo com a imagem abaixo, além de instalar a biblioteca LiquidCristal disponível no link: (https://www.arduinolibraries.info/libraries/liquid-crystal). Ele será responsável por exibir os valores na tela do Display.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/70ce23c5-c3da-4198-8a1a-f2b06a22d894) 
Feito isso, instale o DHT-11 de acordo com a seguinte imagem, inserindo-o no protoboard e baixe a biblioteca dht disponível neste site:(https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-sensor-de-umidade-e-temperatura-dht11). Ele é dispositivo que mede temperatura e umidade. 
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/f4167e07-7cfe-4d9b-a040-96d0d4db02a1) 
Depois, instale o LDR de acordo com a imagem abaixo, que é o responsável por medir a luminosidade.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/ab3739a8-6075-4bc1-98c2-e1a67f420f36)
Agora, o próximo passo é instalar o RTC, para isso, faça como mostra a imagem, além disso, baixe a biblioteca rtc disponível nesse site: (https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-modulo-real-time-clock-rtc-ds1302). Ele servirá como relógio para marcar o tempo de 1 minuto entre as medições.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/21675191-21c1-47e3-a35c-6b455de57ba7) 
Instale também o buzzer de acordo com a imagem, que irá emitir o alerta sonoro dependendo dos resultados.
![image](https://github.com/Bruno872/Sistemas-Embarcados/assets/144634914/c4251f48-950b-4d9e-8bf8-35986afac96b)
Para instalar as bibliotecas no aplicativo arduino, abra o aplicativo, clique em "Sketch", escolha a opção "Incluir Biblioteca" e depois  "Adicionar Biblioteca .ZIP...", selecione o arquivo ZIP da biblioteca baixada e aperte "OK". Siga este procedimento para as três bibliotecas: DHT, RTClib e LiquidCrystal.
Agora, o próximo passo é abrir aplicativo do arduíno no computador e utilizar o seguinte código:

```
#include <dht.h>
 
#include <LiquidCrystal.h> //Carrega a biblioteca LiquidCrystal nativa na IDE

#include <RTClib.h> // Adicione a biblioteca RTClib para trabalhar com o RTC
// Cria um objeto RTC_DS3231 para interagir com o RTC
RTC_DS3231 rtc; //OBJETO DO TIPO RTC_DS3231
 
//DECLARAÇÃO DOS DIAS DA SEMANA
char daysOfTheWeek[7][12] = {"Domingo", "Segunda", "Terça", "Quarta", "Quinta", "Sexta", "Sábado"};
const int ldrPin = A0;
int Pinbuzzer = 10;

//Define os pinos que serão utilizados para ligação ao display
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
 
dht DHT; // Cria um objeto da classe dht
uint32_t timer = 0;
unsigned long time;
 
void setup()
{
  Serial.begin(9600); // Inicializa serial com taxa de transmissão de 9600 bauds
  lcd.begin(16, 2); // Define o display com 16 colunas e 2 linhas
  lcd.clear(); // limpa a tela do display

  pinMode(Pinbuzzer, OUTPUT);

  
if(! rtc.begin()) { // SE O RTC NÃO FOR INICIALIZADO, FAZ
    Serial.println("DS3231 não encontrado"); //IMPRIME O TEXTO NO MONITOR SERIAL
    while(1); //SEMPRE ENTRE NO LOOP
  }
  if(rtc.lostPower()){ //SE RTC FOI LIGADO PELA PRIMEIRA VEZ / FICOU SEM ENERGIA / ESGOTOU A BATERIA, FAZ
    Serial.println("DS3231 OK!"); //IMPRIME O TEXTO NO MONITOR SERIAL
    //REMOVA O COMENTÁRIO DE UMA DAS LINHAS ABAIXO PARA INSERIR AS INFORMAÇÕES ATUALIZADAS EM SEU RTC
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__))); //CAPTURA A DATA E HORA EM QUE O SKETCH É COMPILADO
    //rtc.adjust(DateTime(2018, 9, 29, 15, 00, 45)); //(ANO), (MÊS), (DIA), (HORA), (MINUTOS), (SEGUNDOS)
  }
  delay(100); //INTERVALO DE 100 MILISSEGUNDOS
}
void loop() {  
  
  

  //Executa 1 ves a cada 1m
  if (rtc.now().second() == 0 && millis() - time >= 10000) {

    int contTemp = 0; //variaveis contador 
    int contUmid = 0;
    int contLumi = 0;

    int medTemp = 0; //variaveis media
    int medUmid = 0;
    int medLumi = 0;
  for (int i = 0; i < 5; i++){  
  // Executa 1 vez a cada 2 segundos
  if(millis() - timer>= 2000)
  {
  
    DHT.read11(A1); // chama método de leitura da classe dht,
                    // com o pino de transmissão de dados ligado no pino A1


    // Exibe na serial o valor de luminosidade
    int valorLdr = analogRead (ldrPin);
    int brilho = map(valorLdr, 0, 1023, 0, 100);
    
    Serial.print("Luminosidade = ");
    Serial.print(brilho);
    Serial.println("%");
 
    // Exibe na serial o valor de umidade
    Serial.print("Umidade = ");
    Serial.print(DHT.humidity);
    Serial.print("%");
 
    // Exibe na serial o valor da temperatura
    Serial.print("Temperatura = ");
    Serial.print(DHT.temperature); 
    Serial.println(" Celsius  ");

    int temp = (DHT.temperature);

    int umid = (DHT.humidity);

    contTemp += temp;
    contUmid += umid;
    contLumi += brilho;
        
    timer = millis(); // Atualiza a referência do 5s
  }
  }

  medTemp = contTemp/5;
  medUmid = contUmid/5;
  medLumi = contLumi/5;
  
    if(medTemp < 15 || medTemp > 25)
    {
      digitalWrite(Pinbuzzer, HIGH); 
   }else{
      digitalWrite(Pinbuzzer, LOW);}
      
    if(medLumi > 30)
    {
      digitalWrite(Pinbuzzer, HIGH); 
    }else{
      digitalWrite(Pinbuzzer, LOW);}
      
    if(medUmid < 30 || medUmid > 50)
    {
      digitalWrite(Pinbuzzer, HIGH); 
    }else{
      digitalWrite(Pinbuzzer, LOW);}

   lcd.setCursor(0,1); // Define o cursor na posição de início
    lcd.print("Temp: ");
    lcd.print(medTemp);
    lcd.write(B11011111); // Imprime o símbolo de grau
    lcd.print("C");

        // Exibe no display LCD o valor da umidade
    lcd.setCursor(8,0); // Define o cursor na posição de início
    lcd.print("Lum:");
    lcd.print(medLumi);
    lcd.print("%");



    
    // Exibe no display LCD o valor da umidade
    lcd.setCursor(0,0); // Define o cursor na posição de início
    lcd.print("Umd:");
    lcd.print(medUmid);
    lcd.print("%");

    DateTime now = rtc.now(); //CHAMADA DE FUNÇÃO
    Serial.print("Data: "); //IMPRIME O TEXTO NO MONITOR SERIAL
    Serial.print(now.day(), DEC); //IMPRIME NO MONITOR SERIAL O DIA
    Serial.print('/'); //IMPRIME O CARACTERE NO MONITOR SERIAL
    Serial.print(now.month(), DEC); //IMPRIME NO MONITOR SERIAL O MÊS
    Serial.print('/'); //IMPRIME O CARACTERE NO MONITOR SERIAL
    Serial.print(now.year(), DEC); //IMPRIME NO MONITOR SERIAL O ANO
    Serial.print(" / Dia: "); //IMPRIME O TEXTO NA SERIAL
    Serial.print(daysOfTheWeek[now.dayOfTheWeek()]); //IMPRIME NO MONITOR SERIAL O DIA
    Serial.print(" / Horas: "); //IMPRIME O TEXTO NA SERIAL
    Serial.print(now.hour(), DEC); //IMPRIME NO MONITOR SERIAL A HORA
    Serial.print(':'); //IMPRIME O CARACTERE NO MONITOR SERIAL
    Serial.print(now.minute(), DEC); //IMPRIME NO MONITOR SERIAL OS MINUTOS
    Serial.print(':'); //IMPRIME O CARACTERE NO MONITOR SERIAL
    Serial.print(now.second(), DEC); //IMPRIME NO MONITOR SERIAL OS SEGUNDOS
    Serial.println(); //QUEBRA DE LINHA NA SERIAL
    delay(1000); //INTERVALO DE 1 SEGUNDO
    
  timer = millis(); // Atualiza a referência do 1m
  }
}
```
Caso as entradas utilizadas no arduíno não coincidam com os valores de entrada do código, basta alterar os valores de entrada do código para que eles se adaptem ao seu projeto.

## Uso 
Para o equipameto funcionar, conecte o arduino ao computador, verifique e carregue o código e inicie a compilação, ele calculará as médias dos valores de temperatura, luminosidade e umidade obtidos ao longo de um minuto de medições, realizadas a cada 2 segundos, e as exibirá no display. Caso os valores obtidos estejam fora dos limites estabelecidos pelo programa, o alarme sonoro será ativado através do buzzer.

## Créditos

(https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-sensor-de-umidade-e-temperatura-dht11)
(https://victorvision.com.br/blog/display-lcd-16x2/)
(https://mundoprojetado.com.br/ldr-como-usar/)
(https://blogmasterwalkershop.com.br/arduino/como-usar-com-arduino-modulo-real-time-clock-rtc-ds1302)
(https://mundoprojetado.com.br/buzzer-como-usar-com-o-arduino/#:~:text=O%20circuito%20do%20buzzer%20%C3%A9,um%20pino%20digital%20do%20Arduino.)
(https://www.arduinolibraries.info/libraries/liquid-crystal)
