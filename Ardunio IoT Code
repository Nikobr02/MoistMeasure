/*
  Projektnamn: Moisture Measurer.
  Datum: 19.Nov.2022
  Program: CMAST-1
  Kurskod: MF1001
  Gruppnr: 24
  Studenter i Grupp 24:
    Nikola Obradovic
    Axel Hagman
    Emil Andersson
    Oskar Ardryd
    Daniel Calissendoff
    Viggo Eriksson Wetterskog
  Kod skriven av: Nikola Obradovic
  Examinator: Nihad Subasic

  Fuktsensors productspecifikation : https://wiki.seeedstudio.com/Grove-Moisture_Sensor/
  Inbyggd LEDLampa Infoartikel: https://docs.allthingstalk.com/examples/hardware/get-started-arduino-mkr-wifi/
  Tidsberäkningsinfo : https://forum.arduino.cc/t/using-millis-for-timing-a-beginners-guide/483573
*/
#include "thingProperties.h";
const int luftFuktVarde = 0; //Enl. produktspecification 0-300 är luftfuktighet
const int braFuktVarde = 300; //Enl. produktspecifikation 300-700 är bra värden för fukten i jorden för plantor
const int vattenFuktVarde = 700; ////Enl. produktspecification 700-950 är "vattenfuktighet". I kodens värld är vatten fuktigt.
const int maxFukt = 950;
int sistaBraVarde = 0; // Nu syns tydligt när programmet startas i grafer
String sistaBraMeddelande = "Start av Program"; // Nu syns det tydligt i chatten när programmet startas
CloudPercentage sistaBraFukt;
unsigned long startTid = 0;  // Sparar senaste tiden vi körde koden
unsigned long tidNu;
unsigned long interval = 3600000;  // interval på hur ofta vi tar värden och gör något med det (milliseconds)
int i =0;
void setup() {
  //Initialiserar serial och ger den lite tid att öppna porten.
  Serial.begin(9600);
  delay(1500);

  //Den inbyggda LEDn har analog RGB input.
  WiFiDrv::pinMode(25, OUTPUT); //definierar inbyggda LEDns gröna LEDpin
  WiFiDrv::pinMode(26, OUTPUT); //definierar inbyggda LEDns röda LEDpin
  WiFiDrv::pinMode(27, OUTPUT); //definierar inbyggda LEDns blåa LEDpin

  // Lägger till våra onlinevaribler som vi kan läsa molnet med thingProperties.h
  initProperties();

  // Ansluter till Arduino IoT Cloud
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);

  //Följande skriver ut felmeddleanden till kopplingen om Arduinon inte lyckas ansluta till Serial.
  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();
}

void loop() {
  if (sensorVarde !=0 || i==0){
    ArduinoCloud.update(); //Skriver ut datan till IoT Cloud. Måste göras ofta annars förloras kontakten.
  }
  tidNu = millis(); // Tiden som gått från programmet starta till nu
  if ((tidNu - startTid >= interval && ArduinoCloud.connected()) || i == 0 ) {
    i=1;
    startTid = tidNu; // Sparar senaste tiden vi körde koden    
    sensorVarde = analogRead(A0); //Läser infon från vår fuktsensor som är ansluten till A0 poten.
    if ((sensorVarde >5)){ //Ibland blir det fel och sensorvärdet orimligt lågt, då ska vi strunta i det värdet
      //Konverterar sensorVardet till en procentdsats genom att sätta luftfuktigheten som bottenvärdet och vattenfuktighet som maxvärdet
      fuktProcent = map(sensorVarde, luftFuktVarde, vattenFuktVarde, 0, 100);
      // Printar infon till Serial.
      Serial.print("Fuktigheten är: ");
      Serial.print(fuktProcent);
      Serial.print("%. ");
      Serial.print(sensorVarde);
      // Är fuktnivån mindre än luftfuktighetens maxvärde, skriv till användaren att den bör vattna till IoT Cloud och sätt på LEDn på röd färg.
      if (sensorVarde < braFuktVarde)
      {
        meddelande = " Fuktnivån är låg. Du bör vattna.";
        WiFiDrv::analogWrite(25, 0);
        WiFiDrv::analogWrite(26, 255); //Sätter Lampan på Röd färg om den behöver vattnas
        WiFiDrv::analogWrite(27, 0);
        interval = 900000; //Börjar mäta var 15e minut så man snabbare kan se om man lagt till lagom mängd vatten
      }
      // Är fuktnivån mer än luftfuktigheten och mindre än fuktigheten för vatten, Släck LEDn och skriv att fuktnivån är OKej till IoT Cloud.
      else if ((sensorVarde > braFuktVarde) && (sensorVarde < vattenFuktVarde) )
      {
        meddelande = " Fuktnivån är okej.";
        WiFiDrv::analogWrite(25, 0);
        WiFiDrv::analogWrite(26, 0); //Stänger av lampan om fukten är OK
        WiFiDrv::analogWrite(27, 0);
      }
      // Är fuktnivån mer än vattens fuktighetsmaxvärde, skriv till användaren att den bör kolla till sensorn till IoT Cloud och sätt på LEDn på blå färg.
      else if (sensorVarde > vattenFuktVarde) {
        meddelande = " VARNING. Orimligt högt fuktvärde angavs, kontrollera att en pöl med vatten inte skapats i din kruka";
        WiFiDrv::analogWrite(25, 0);
        WiFiDrv::analogWrite(26, 0); //Lyser Blått om Udda värde angavs
        WiFiDrv::analogWrite(27, 255);
      }
      // Är fuktnivån nånting annat oväntat, skriv till användaren att den bör kolla till sensorn till IoT Cloud och sätt på LEDn på blå färg.
      else {
        meddelande = " VARNING, Oväntat värde angavs av fuktsensorn. Kontrollera att den är rätt monterad";
        WiFiDrv::analogWrite(25, 0);
        WiFiDrv::analogWrite(26, 0); //Lyser Blått om Udda värde angavs
        WiFiDrv::analogWrite(27, 255);
      }  
      Serial.println(meddelande);
    
      sistaBraVarde = sensorVarde;
      sistaBraMeddelande = meddelande;
      sistaBraFukt = fuktProcent;
    }
  }
  sensorVarde = sistaBraVarde;
  meddelande = sistaBraMeddelande;
  fuktProcent = sistaBraFukt;
}
