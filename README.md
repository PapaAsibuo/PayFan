# PayFan
Here is the code to create PayFan: A fan that runs based on money credited onto a student ID card 
/*
   --------------------------------------------------------------------------------------------------------------------
   Example sketch/program showing how to read new NUID from a PICC to serial.
   --------------------------------------------------------------------------------------------------------------------
   This is a MFRC522 library example; for further details and other examples see: https://github.com/miguelbalboa/rfid

   Example sketch/program showing how to the read data from a PICC (that is: a RFID Tag or Card) using a MFRC522 based RFID
   Reader on the Arduino SPI interface.

   When the Arduino and the MFRC522 module are connected (see the pin layout below), load this sketch into Arduino IDE
   then verify/compile and upload it. To see the output: use Tools, Serial Monitor of the IDE (hit Ctrl+Shft+M). When
   you present a PICC (that is: a RFID Tag or Card) at reading distance of the MFRC522 Reader/PCD, the serial output
   will show the type, and the NUID if a new card has been detected. Note: you may see "Timeout in communication" messages
   when removing the PICC from reading distance too early.

   @license Released into the public domain.

   Typical pin layout used:
   -----------------------------------------------------------------------------------------
               MFRC522      Arduino       Arduino   Arduino    Arduino          Arduino
               Reader/PCD   Uno/101       Mega      Nano v3    Leonardo/Micro   Pro Micro
   Signal      Pin          Pin           Pin       Pin        Pin              Pin
   -----------------------------------------------------------------------------------------
   RST/Reset   RST          9             5         D9         RESET/ICSP-5     RST
   SPI SS      SDA(SS)      10            53        D10        10               10
   SPI MOSI    MOSI         11 / ICSP-4   51        D11        ICSP-4           16
   SPI MISO    MISO         12 / ICSP-1   50        D12        ICSP-1           14
   SPI SCK     SCK          13 / ICSP-3   52        D13        ICSP-3           15
*/

#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 53
#define RST_PIN 5

MFRC522 rfid(SS_PIN, RST_PIN); // Instance of the class

MFRC522::MIFARE_Key key;

/* Enter the UID for your card here! Use the "Display Card UID" program to determine the ID on your Lehigh ID card and on the fob that came with your kit.
 Replace the numbers following the 0x in the variable YOUR_UID with the numbers from "Display_Card_UID" code.
  For example, if your card ID is: 4F 35 28 3D, line 46 would become:
    #define YOUR_UID {0x4F, 0x35, 0x28, 0x3D}
    That is all you need to change for the code to work
    Be careful with the characters in the UID. There is no letter O. That is, 0 (zero) not O (Oh)
 */
 //Servo myservo;
#define YOUR_UID {0xB6, 0x77, 0x93, 0x76}    //     <------ Change this line per instructions above
byte YourUid[] = YOUR_UID;

// Init array that will store new NUID
byte nuidPICC[4];

bool validCard = false;
bool cardScanned = false;
int cardcounter = 0;

int motorPin = 9;              // Declares the pin the motor is connected to

void setup() {
 
  Serial.begin(9600);
  SPI.begin(); // Init SPI bus
  rfid.PCD_Init(); // Init MFRC522

  for (byte i = 0; i < 6; i++) {
    key.keyByte[i] = 0xFF;
  }

  Serial.println(F("Scan your Lehigh ID card to use the PayFan."));
}




void loop() {

  ReadCard();

  validCard = compareUID(nuidPICC, YourUid); // will be true or false
  
  if ( validCard == true && cardScanned == true ) {   
    granted();
  }
  else if( validCard != true && cardScanned == true ) {
    denied();
  }
  else{
  //Serial.println("Waiting for Card . . ."); 
  delay(100);
  }

}




void ReadCard(){
   // Reset the loop if no new card present on the sensor/reader. This saves the entire process when idle.
  if ( ! rfid.PICC_IsNewCardPresent())
    return;
  
  // Verify if the NUID has been readed
  if ( ! rfid.PICC_ReadCardSerial())
    return;

  MFRC522::PICC_Type piccType = rfid.PICC_GetType(rfid.uid.sak);

  // Check is the PICC of Classic MIFARE type
  if (piccType != MFRC522::PICC_TYPE_MIFARE_MINI &&
      piccType != MFRC522::PICC_TYPE_MIFARE_1K &&
      piccType != MFRC522::PICC_TYPE_MIFARE_4K) {
    Serial.println(F("Your tag is not of type MIFARE Classic."));
    return;
  }


  // Store NUID into nuidPICC array
  for (byte i = 0; i < 4; i++) {
    nuidPICC[i] = rfid.uid.uidByte[i];
  }

  

//  Serial.print(F("\n\nThe card UID is:"));
  //printHex(rfid.uid.uidByte, rfid.uid.size);
 // Serial.println();
  cardScanned = true;

//  if ( compareUID(nuidPICC, newUid) ) {   // Calls function as condition in if statement, function will output true or false
//    granted();
//  }
//  else {
//    denied();
//  }

  // Halt PICC
  rfid.PICC_HaltA();

  // Stop encryption on PCD
  rfid.PCD_StopCrypto1();
}



void granted() {
  Serial.println("\nEnjoy!\n");
   // Do whatever else you want to when access is granted
   analogWrite(motorPin, 255);
  delay(20000);  // Fan runs for 20 seconds
  analogWrite(motorPin, 0);
  cardcounter = cardcounter + 1;            // This counter determines how many times the card is used
  if (cardcounter > 19){
    Serial.println("\nYou are out of Fanbucks\n");    // If the user has used all 20 fanbucks an error messsage is printed
  }
  
  cardScanned = false;
 }

void denied() {
  Serial.println("\nThis card is not supported for PayFan\n");
  // Do whatever else you want to when access is denied
   delay(1000); 
    analogWrite(motorPin, 0);
  
  cardScanned = false;
  }

/**
   Helper routine to dump a byte array as hex values to Serial.
*/
void printHex(byte *buffer, byte bufferSize) {
  for (byte i = 0; i < bufferSize; i++) {
    Serial.print(buffer[i] < 0x10 ? " 0" : " ");
    Serial.print(buffer[i], HEX);
  }
}


///////////////////////////////////////// Compare UIDs   ///////////////////////////////////
bool compareUID ( byte a[], byte b[] ) {
  for ( uint8_t k = 0; k < 4; k++ ) {   // Loop 4 times
    if ( a[k] != b[k] ) {     // IF a != b then false, because: one fails, all fail
      return false;
    }
  }
  return true;
}
