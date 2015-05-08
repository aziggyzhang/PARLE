# PARLE
Parallel Acoustics for Renovating Library Environment

//..............................................................................
// Huge thanks to Mohannad Rawashdeh
// for MP3 1.2V Shield test .

#include <SoftwareSerial.h>
SoftwareSerial Geno(7,8); // Rx , Tx
unsigned char Data[10];
unsigned char i;

#define PIN_GATE_IN 2
#define IRQ_GATE_IN  0
#define PIN_LED_OUT 13
#define PIN_ANALOG_IN A0

int vol = 5; 

void soundISR()
{
  int pin_val;

  pin_val = digitalRead(PIN_GATE_IN);
  digitalWrite(PIN_LED_OUT, pin_val);   
}

void setup() {
  
  Serial.begin(9600);
  pinMode(PIN_LED_OUT, OUTPUT);
  pinMode(PIN_GATE_IN, INPUT);
  attachInterrupt(IRQ_GATE_IN, soundISR, CHANGE);
  Serial.println("Initialized");
  
  // put your setup code here, to run once:
  Geno.begin(9600);
  //SetVolume(vol);
  
  
}


char soundDetectorState(){
  
  int sound;

  // Check the envelope input
  sound = analogRead(PIN_ANALOG_IN);

  // Convert envelope value into a message
  if(sound <= 30)
  {
    return 'Q';
  }
  else if( (sound > 30) && ( sound <= 40) )
  {
    return 'M';
  }
  else if(sound > 40)
  {
    return 'L';
  }
}



void loop() {
  
  Geno.begin(9600);
  FileSource('U_Disk',0x05);
  SetPlayMode('Single_play');
  
  char c;
  c = soundDetectorState();
  Serial.println(c);
  
  switch(c) {
    case 'Q':  
        for(int j = 16; j > 8 ; j--){
          SetVolume(j); // set volume from 0-31
          //delay(200);
        }
        for (int x = 8; x < 16; x ++){
          SetVolume(x);
          //delay(200);
        }
        break;
    case 'M':
      /*for (int l = 14; l < 17; l++){
        SetVolume(l);
        delay(300);
      }
      for (int y = 17; y > 14; y--){
        SetVolume(y);
        delay(300);
      }*/
        SetVolume(16);
        delay(800);
        break;
    case 'L':
        for(int v = 16 ; v < 22; v++){
            SetVolume(v); // set volume from 0-31
            delay(500);
        }
        for (int u = 22; u > 16; u--){
          SetVolume(u);
          delay(1000);
        }
        break;   
    }
  
  
 
}
// Select File sorce "SD Card, SPI Flash , U Disk "
void FileSource( char type, byte track ){
  play_pause();// Pause
   Data[0] = 0x7E; 
   Data[5] = 0x7E;   
  switch (type){
   case 'SD_card':
          // START
    Data[1] = 0x04;          // Length
    Data[2] = 0xA0;          // Command
    Data[3] = 0x00;          // file number high byte
    Data[4] = track;          // file number low byte 
    break;
    case 'SPI_Flash':
    Data[1] = 0x04;          // Length
    Data[2] = 0xA1;          // Command spi flash 0XA1
    Data[3] = 0x00;          // file number high byte
    Data[4] = track;
   break;
    case 'U_Disk':
    Data[1] = 0x04;          // Length
    Data[2] = 0xA2;          // Command
    Data[3] = 0x00;          // file number high byte
    Data[4] = track;          // file number low byte 
    break;
 
  }
  Command(Data,5);
  play_pause();// Pause
}

void SetVolume( int vol){
   Data[0] = 0x7E;          // START
   Data[1] = 0x03;          // Length Not 0x02
   Data[2] = 0xA7;          // Command
   Data[3] = vol;          // new volume
   Data[4] = 0x7E;          // END
   Command(Data,5);
}

void SetPlayMode(char type){
   Data[0] = 0x7E;          // START
   Data[4] = 0x7E;          // START
   switch (type){
   case 'Single_play': 
   Data[1] = 0x02;          // Length
   Data[2] = 0xA9;          // Command
   Data[3] = 0x00;          //Mode parameter
   break;
   case 'Repeat_single': 
   Data[1] = 0x02;          // Length
   Data[2] = 0xA9;          // Command
   Data[3] = 0x01;          //Mode parameter
   break;
   case 'Repeat_all': 
   Data[1] = 0x02;          // Length
   Data[2] = 0xA9;          // Command
   Data[3] = 0x02;          //Mode parameter
   break;
   case 'Play_Random': 
   Data[1] = 0x02;          // Length
   Data[2] = 0xA9;          // Command
   Data[3] = 0x03;          //Mode parameter
   break;
}
Command(Data,5);
}
//...............................

void play_pause(){
  Data[0] = 0x7E;          // START
  Data[1] = 0x02;          // Length
  Data[2] = 0xA3;          // Command
  Data[3] = 0x7E;          //Mode parameter
  Command(Data,4);
}

  void Stop(){
  Data[0] = 0x7E;          // START
  Data[1] = 0x02;          // Length
  Data[2] = 0xA4;          // Command
  Data[3] = 0x7E;          //Mode parameter
  Command(Data,4);
}

  void Next(){
  Data[0] = 0x7E;          // START
  Data[1] = 0x02;          // Length
  Data[2] = 0xA5;          // Command
  Data[3] = 0x7E;          //Mode parameter
  Command(Data,4);
}
  void Previous(){
  Data[0] = 0x7E;          // START
  Data[1] = 0x02;          // Length
  Data[2] = 0xA6;          // Command
  Data[3] = 0x7E;          //Mode parameter
  Command(Data,4);
}

void Command(unsigned char *Data, int length){
    for(int i=0; i<length; i++){
    Geno.write(Data[i]);
    }
 
}
