# Sync-Drone-Nodes
// created by Ahmad Abdullah (German Jordanian University WSN project, supervised 
// by Dr. Ala' Khalifeh)

//The programming code below shows the implementation of a Wireless communication between two nodes in the WSN Network. Firstly ,the sync node which consist of 
// a waspmote microcontroller, xbee radio model and APC220 High Radio frequency transceiver. Secondly the drone node which consist of a waspmote microcontroller and 
// a APC220 High Radio frequency transceiver. 

// Application Description: we implemented a WSN where the sync node is supposed to collect all the data from the WSN network store it in a SD card, the method 
// of communication within the network is the xbee radio model,  when the drone is within the range of communication of the WSN the sync node supposed to send the 
// data to the drone using APC220 High Radio frequency transceiver, when the data reaches the drone node it is stored on a SD card 




 // Sync Node Waspmote Code 
 
#include <WaspXBeeDM.h> // Xbee radio models library
#include <WaspFrame.h>
#define DEF_BAUD_RATE 9600 //Define the BAUD Rate which will be used for the communication with the drone node 
char filename[]="FILE2.TXT"; //Define the text file name where the data will be stored in the SD card 
uint8_t sd_answer;
int32_t numLines=0; // Integer counter which will be used later to count the number of frames stored in the SD Card
uint8_t error; //Error is integer variable that will be used to check if we are facing any  errors in receiving the packets from nearby nodes
uint8_t answer[256];
char response[256];
char x[256];
bool conf = false; //Boolian Variable that will be used later to control the communication between the nodes
bool flag1 = false;
int i,f,n,c = 0;
int m =-1;

// Setup the Waspmote Micontroller

void setup()
{
  settingup(); 
}

// Main Part of the code (Operantional Part)

void loop()
{
  if(!conf)
  {
  
  error = xbeeDM.receivePacketTimeout( 3000 ); //Check and receives packers from the nearby nodes 

  // check what data we received from the other nodes   
  if( error == 0 ) 
  {
    // Print the data received from other nodes and stored in the buffer 
    
    USB.print(F("Data: "));  
    USB.println( xbeeDM._payload, xbeeDM._length);
    
    // Print the lengthe of data received 
    
    USB.print(F("Length: "));  
    USB.println( xbeeDM._length,DEC);
    
    //Save the data received in array
    for(uint8_t i = 0; i < xbeeDM._length; i++)
    {  
     x[i] = xbeeDM._payload[i]; 
     }
    // Save the data in the SD File   
     
    sd_answer = SD.appendln(filename,x); 
  }
  
   // This else refers the if statment of the error, if the error wasn't 0 means that we have a packet timeout in the network and we couldn't receive the packet
   therefore prinited the following message to indicate the error 
   else
  {
    // Print error message:
    /*
     * '7' : Buffer full. Not enough memory space
     * '6' : Error escaping character within payload bytes
     * '5' : Error escaping character in checksum byte
     * '4' : Checksum is not correct    
     * '3' : Checksum byte is not available 
     * '2' : Frame Type is not valid
     * '1' : Timeout when receiving answer   
    */
    USB.print(F("Error receiving a packet:"));
    USB.println(error,DEC);     
  }
 
  //show  the sd card file
  SD.showFile(filename);
  delay(200);
  numLines = SD.numln(filename); // Indicator of how many frames stored in the SD Card
  // Reset the buffer to ensure that the sync node is always searching for the drone node 
  serialFlush(UART1); 
  memset(answer, 0x00, sizeof(answer) ); 
  // Print the number of frames stored in the SD card
  USB.println("number of stored frames is :  ");
  USB.println(numLines);
  //Start looking for the drone only after we saved 60 or more packets in the sd card successfully
  if(numLines > 60)
  {
  receivereq(); 
  }

 //Initial assumption that the drone, therefore we need to check if the drone has arrived 
 if(conf){
   m++;
   sendframe();
   receiveack();
   delay(1500);  
   if(m==numLines){
     sendLastframe();
     m=-1;
     conf=false; 
     sleepnode();
   }
  }
}


//Looking for the drone node- start thr handshaking process 

void receivereq()
{

  Utils.setMuxAux2(); //aux1 for hc12, aux2 for hf
  delay(500);
  USB.println("listening for 12s...");
  memset(answer, 0x00, sizeof(answer) );
  i=0;
  c=0;
  while(!serialAvailable(1) && c < 120)
  {
  delay(100);
  c++;
  }

  //Read the UART1 buffer, connected to aux2(hf)
  while(serialAvailable(1)) 
  {
    answer[i]=serialRead(1);
    i++;
    delay(10);
  }
  for (uint8_t a = 0; a < i; a++)
  {
    USB.print(answer[a]); 
  }
  USB.println();
  for(uint8_t o = 0; o < i; o++) 
  {
  response[o] = (char)answer[o];
  }
  //check  if drone was detected 
  if(response[0] == '1' && response[1] == '2' && response[2] == '3' && response[3] == '4')
  { 
  USB.println("pass matched, drone detected!");
  conf = true;   //change states, drone is detected 
  Utils.setMuxAux1(); //turn the uart to hc12
  response[0] = '\0'; //reset response to empty string in order to prepare to receive ack from drone node
  delay(500);
 //send response back to the drone 3 times, 2sec between each time
  f or(int f = 0; f < 3; f++)
  { 
    printString("4321\n", 1);
    USB.println("4321");
    delay(2000); 
  }
  }
  delay(1500);
}

// Send frames to the drone node 

void sendframe()

{
    Utils.setMuxAux1(); 
    delay(100);
    // read new line (it is stored in SD.buffer)
    SD.catln(filename,m,1);
    printString(SD.buffer, 1); //send the frame from the sd card to drone node
     delay(1200);

}

// Send the last frames to the drone node 
void sendLastframe()
{
    Utils.setMuxAux1(); 
    delay(500);
    printString("last frame was sent from network 1, going to sleep ...", 1); //send this string to
    USB.println("last frame was sent from network 1, going to sleep ...");  //indicate that we finished transmission

}


void receiveack()
{
   Utils.setMuxAux2(); //aux1 for hc12, aux2 for hf
   delay(500);
   USB.println("listening...");
   memset(answer, 0x00, sizeof(answer) );
   i=0;
   n=0;
   while(!serialAvailable(1) && n < 70){
   delay(100);
   n++;
}

USB.println(n);

while(serialAvailable(1))
{
   answer[i]=serialRead(1);
   i++;
   delay(10);
}

USB.println("response as bytes: ");

for (uint8_t a = 0; a < i; a++)
{
    USB.print(answer[a]); //typing out the response
}

USB.println();
//response[0] = '\0';

USB.println("response as chars: ");

//convert the response from bytes to chars

for(uint8_t o = 0; o < i; o++) 
{
response[o] = (char)answer[o];
}

USB.println(response);

if(n==70 || (response[0] == '1' && response[1] == '2' && response[2] == '3' && response[3] == '4'))
{
  conf=false;
  USB.println("timeout 7s reached or drone in wrong loop switching back to listening ");
}
response[0] = '\0';
}


//Microcontroller setup function 

void settingup(){
 
  RTC.ON();
  ACC.ON();
  USB.ON();
  PWR.powerSocket(UART1, HIGH); //powering up
  PWR.setSensorPower(SENS_3V3, SENS_ON); //set hc12 and hf on
  WaspUART uart = WaspUART();
  uart.setUART(UART1);
  delay(500);
  uart.setBaudrate(9600);
  delay(500);
  Utils.setMuxAux2(); //aux1 for hc12, aux2 for hf
  delay(500);
  serialFlush(UART1);
  uart.beginUART();
  delay(1500);
  SD.ON();
  // Delete the old files
  sd_answer = SD.del(filename);
  if( sd_answer == 1 )
  {
  USB.println(F("file deleted"));
  }
 else
 {
 USB.println(F("file NOT deleted"));
 }
 // Create new file
 sd_answer = SD.create(filename);
 if( sd_answer == 1 )
 {
 USB.println(F("file created"));
 }
 else
 {
 USB.println(F("file NOT created"));
 }
 xbeeDM.ON();
 xbeeDM.writeValues();
 }
 
//Sleep mode function

void sleepnode()
{
 USB.println("finished sending sd card file, entering deep sleep mode");
 PWR.deepSleep("00:00:00:17", RTC_OFFSET, RTC_ALM1_MODE1,
 SENSOR_ON);
 USB.ON();
 USB.println(F("wake up\n"));
 if (intFlag & RTC_INT)
 {
 USB.println(F("-------------------------------------"));
 USB.println(F("RTC INT captured"));
 USB.println(F("-------------------------------------"));
 // clear flag
 intFlag &= ~(RTC_INT);
settingup();
 }
}


// DRONE NODE CODE
// DRONE NODE CODE

// including the SD library

#include <SD.h>
#include <SPI.h>

String Data; // Later on this string will be used to store data on 
File file;  // Create a file on the drone SD card  
char myFileName[] = "test.txt"; // Name the file on the SD card 
String pass = "1234\n";// Password used for the handshaking process 
String Data2;
String Data3;
String Data4;
String ack = "ACK\n";
int conf = 0;
int m = 0;

// Set Up the drone micro controller  
void setup() {
pinMode(10, OUTPUT);  
digitalWrite(10, HIGH);
Serial3.begin(9600); //hc12 link
Serial2.begin(9600); //hf link
Serial.begin(9600); //Usb 
pinMode(53, OUTPUT); //slave select for sd card
digitalWrite(53, LOW); //making sure sd card is selected
pinMode(SS, OUTPUT); // Sd card Pin
if(!SD.begin(53))Serial.println("SD card not initialized");  //incase sd card is not working
else Serial.println("SD card initialized");
}
void loop()
{
if(conf==0)sendandlisten();  //drone searching for sync nodes

if(conf==1){
ReadFrameandSendAck(); //receiving from sync node
delay(3500);
if(m==3){
  conf=0; 
  m=0;
  file.close(); //close file after reception is complete
}
}
}

// Listening to the network-Searching for the network

void sendandlisten()
{
Serial2.print(pass); 
Serial.println(pass);
delay(2000);    
Data2 = Serial3.readString();
Serial.println("received...");
Serial.print(Data2);
//  Detecting the Network
if(Data2 == "4321\n") 
  {                      
   Serial.println("Correct response, network detected");
   file = SD.open(myFileName, FILE_WRITE);    //open a file
   file.println("----new initialization---"); //add a new line indicating a new session
   file.flush(); //save the file
   conf++; //change state to network not found to network found
  }   
 }

// communication process between the sync node and drone node 
void ReadFrameandSendAck()
{
if(Serial3.available()){   
delay(200);
Data4 = Serial3.readString();
Serial.println("from HF: ");
Serial.println(Data4);
file.println(Data4);  
file.flush();  
Serial.println("sending ack "); 
delay(500);
Serial2.print(ack); 
m = 0;
}
else {
  m++; //we stopped receiving frames, at m=3 we'll go back to sending requests
  Serial.println("no new frame received ");
}
}


// Matlab Parsing code to parse the frames (data) collected by the drone in the SD card

function [t] = parse(a) 
sz = size(a);
edit exp.txt;
fileID = fopen('exp.txt','w');
for i=1:1:sz(2)
a1 = string(a(1,i));
           str1 = extractBetween(a1,'TC:','#HUM');
           str2 = extractBetween(a1,'HUM:','#PRES');
           str3 = extractBetween(a1,'PRES:','#PIR');
           str4 = extractBetween(a1,'PIR:','#');
           fprintf(fileID,'Temperature : ');
           fprintf(fileID,'%s \n',str1);
           fprintf(fileID,'Humidity : ');
           fprintf(fileID,'%s \n',str2);
           fprintf(fileID,'Pressure : ');
           fprintf(fileID,'%s \n',str3);
           fprintf(fileID,'Motion detection : ');
           fprintf(fileID,'%s \n',str4);
           fprintf(fileID,'--------------------------- \n');
end
fclose(fileID);
return

 
