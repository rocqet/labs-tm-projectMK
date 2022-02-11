# Marcel Król
# Projekt Technika Mikroprocesorowa

## Temat projektu:
Czujnik ruchu reagujący alarmem dźwiękowym
#### Lista elementów
- Arduino Uno
- Buzzer
- Czujnik ruchu 
- Przewody
- Płytka prototypowa
- Wyświetlacz
#### Kod:
```
#include <LiquidCrystal_I2C.h>

int calibrationTime = 15;
LiquidCrystal_I2C lcd(0x27, 16, 2); //definicja ekranu 16x2

//czas w ktorym sensor nadaje stan niski
long unsigned int lowIn;         

//czas w milisekundach w ktorym sensor jest w stanie niskim
//przed wykryciem braku ruchu
long unsigned int pause = 5000;  

boolean lockLow = true;
boolean takeLowTime;  

int pirPin = 3;    //pin polaczony z wyjsciem sensora PIR
int ledPin = 13;   //dioda na płytce
int buzzer = 9;    //sygnał buzzera 


/////////////////////////////
//SETUP
void setup(){
  Serial.begin(9600);
  pinMode(pirPin, INPUT);
  pinMode(ledPin, OUTPUT);
  pinMode(buzzer, OUTPUT);
  digitalWrite(pirPin, HIGH);
  digitalWrite(buzzer, HIGH);
  lcd.init(); //inicjalizacja ekranu
  lcd.backlight(); //włączenie podświetlenia ekranu

  //dajemy czas na kalibracje sensora
  lcd.clear(); //czyszczenie ekranu
  lcd.setCursor(0, 0); //ustawienie kursora w pozycji 0, 0
  Serial.print("calibrating sensor ");
  lcd.print("Trwa kalibracja");
  lcd.setCursor(0,1);
  lcd.print("czujnika");
    for(int i = 0; i < calibrationTime; i++){
      Serial.print(".");
      delay(1000);
      }
    Serial.println(" done");
    lcd.clear(); //czyszczenie ekranu
    lcd.setCursor(0, 0); //ustawienie kursora w pozycji 0, 0
    lcd.print("Udana");
    Serial.println("SENSOR ACTIVE");
    delay(1000);
 

////////////////////////////
//LOOP
void loop(){
  lcd.clear(); //czyszczenie ekranu
  lcd.setCursor(0, 0); //ustawienie kursora w pozycji 0, 0

     if(digitalRead(pirPin) == HIGH){
       digitalWrite(ledPin, HIGH);
       lcd.print("Wykryto ruch");
       digitalWrite(buzzer, LOW); //przypisanie stanu niskiego na wejściu buzzera
       delay(100);//dioda led wizualizuje stan na wyjsciu sensora
       digitalWrite(buzzer, HIGH);
       if(lockLow){  
         //upewniamy sie ze odczekalismy wystarczajaca ilosc czasu na zmiane stanu
         lockLow = false;            
         Serial.println("---");
         Serial.print("motion detected at ");
         Serial.print(millis()/1000);
         Serial.println(" sec"); 
         delay(50);
         }         
         takeLowTime = true;
       }

     if(digitalRead(pirPin) == LOW){       
       digitalWrite(ledPin, LOW);  //dioda led wizualizuje stan na wyjsciu sensora

       if(takeLowTime){
        lowIn = millis();          
        takeLowTime = false;       
        }
       //jesli sensor jest w stanie niskim dluzej niz ustalony przez nas czas
       //to zakladamy ze sensor nie wykryje juz ruchu
       if(!lockLow && millis() - lowIn > pause){  
           lockLow = true;                        
           Serial.print("motion ended at ");      //output
           Serial.print((millis() - pause)/1000);
           Serial.println(" sec");
           delay(50);
           }
       }
  }
}
```

