
Code for transmitter module:
#include <nRF24L01.h>
#include <printf.h>
#include <RF24.h>
#include <RF24_config.h>

#include<Wire.h>
#include<SPI.h>
#include<nRF24L01.h>

int CE_pin=7, CSN_pin=8;
RF24 myRadio(CE_pin,CSN_pin);
const byte pipe [6] = "00001";

const int MPU_addr=0x68; int16_t AcX,AcY,AcZ,Tmp,GyX,GyY,GyZ;

int minVal=265; int maxVal=402;

double x; double y; double z;

void setup(){ 
  Wire.begin(); 
  Wire.beginTransmission(MPU_addr); 
  Wire.write(0x6B); Wire.write(0); 
  Wire.endTransmission(true); 
  Serial.begin(9600);
  myRadio.begin();  
  myRadio.setChannel(115); 
  myRadio.setPALevel(RF24_PA_MAX);
   
  myRadio.openWritingPipe(pipe);
  myRadio.stopListening();
  delay(1000);
  } 
void loop(){ 
  int res=0;
  Wire.beginTransmission(MPU_addr); 
  Wire.write(0x3B); 
  Wire.endTransmission(false); 
  Wire.requestFrom(MPU_addr,14,true); 
  AcX=Wire.read()<<8|Wire.read(); 
  AcY=Wire.read()<<8|Wire.read(); 
  AcZ=Wire.read()<<8|Wire.read(); 
  int xAng = map(AcX,minVal,maxVal,-90,90); 
  int yAng = map(AcY,minVal,maxVal,-90,90); 
  int zAng = map(AcZ,minVal,maxVal,-90,90);

  x= RAD_TO_DEG * (atan2(-yAng, -zAng)+PI);
  y= RAD_TO_DEG * (atan2(-xAng, -zAng)+PI);
  z= RAD_TO_DEG * (atan2(-yAng, -xAng)+PI);

  //Serial.print("AngleX= "); Serial.println(x);

 // Serial.print("AngleY= "); Serial.println(y);

  //Serial.print("AngleZ= "); Serial.println(z);
  delay(8000); 

  if (x >= 90 && x <=100 )
  {
   res=1;
  }
  
  myRadio.write(&res,sizeof(res));
  Serial.println(res);
}
  
Code for receiver module:

#include<SPI.h>
#include<nRF24L01.h>
#include<RF24.h>
#include <LiquidCrystal.h>
/* Create object named lcd of the class LiquidCrystal */
LiquidCrystal lcd(13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3);  /* For 8-bit mode */
//LiquidCrystal lcd(13, 12, 11, 6, 5, 4, 3);  /* For 4-bit mode */

unsigned char Character1[8] = { 0x04, 0x1F, 0x11, 0x11, 0x1F, 0x1F, 0x1F, 0x1F }; /* Custom Character 1 */
unsigned char Character2[8] = { 0x01, 0x03, 0x07, 0x1F, 0x1F, 0x07, 0x03, 0x01 }; /* Custom Character 2 */
int CE_pin=7, CSN_pin=8;
RF24 myRadio(CE_pin,CSN_pin);
const byte pipe [6] = "00001";
void setup()
{
  lcd.begin(16,2);  /* Initialize 16x2 LCD */
  lcd.clear();  /* Clear the LCD */
  lcd.createChar(0, Character1);  /* Generate custom character */
  lcd.createChar(1, Character2);
  Serial.begin(9600);
  delay(1000);
  myRadio.begin();  
  myRadio.setChannel(115); 
  myRadio.setPALevel(RF24_PA_MIN);
  myRadio.openReadingPipe(0,pipe);
  myRadio.startListening();
  delay(1000);
}
void loop()
{
  if (myRadio.available())
  {   int res=0;
    myRadio.read(&res,sizeof(res));
    Serial.println("printing ");
    Serial.println(res);
  lcd.setCursor(0,0); /* Set cursor to column 0 row 0 */
  lcd.print("hello!!!");/* Print data on display */
  lcd.setCursor(0,1);
 switch (res)
 {
  case 1:
  lcd.print("water needed");
  Serial.print("ok");
  break;
  case 2:
  lcd.print("food needed");
  break;
  case 3:
  lcd.print("fan on/off");
  break;
  case 4:
  lcd.print("call doctor");
  break;
  default:
  lcd.print("out");
  break;
 }
}
}
