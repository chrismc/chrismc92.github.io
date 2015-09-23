---
layout: post
title: "Big Ole Watch"
quote: "The quest for the dorkiest timepiece"
image: /media/2015-09-23-Big-Ole-Watch/InitializedDisplay.jpg
video: false
comments: true
dark: false
---
I've had my eye on the [Sparkfun 7-Segment Serial Display](https://www.sparkfun.com/products/11440) for some time now, but never could come up with a suitable project for it. A good friend of mine told me they were throwing together a mad scientist costume, and when they asked me if I could make them a watch in that theme, I immediately grabbed one of the displays.

The display is marketed as a quick and easy solution to projects where you want a seven segment display, but don't want to spend the time wiring up all those pins. It accepts commands serially using TTL, SPI, or I2C. That's pretty cool, but even though I'm making the watch a bit exaggerated and oversized, I didn't really want to include a full microcontroller circuit in the housing. Thankfully, the brains behind interpreting the serial data and sending the appropriate commands to the display is an ATMega328p with none other than the Arduino bootloader! 

The pins required for FTDI communication are broken out on the board. I popped some straight through headers onto the board, and soldered one of the pins down to keep it in place (the other headers were a bit close to the actual display, which I was afraid of melting). It seemed a good enough connection for the prototyping stage. Then all that was left was to take an Arduino board I had lying around and pop the MCU out, allowing me to use the board's built in FTDI to USB converter to talk to the ATMega onboard the display.

{% include image.html url="/media/2015-09-23-Big-Ole-Watch/FTDIWiring.jpg" width="50%" description="Wired up for programming" %} 

Finding the files to place in the Arduino hardware directory for this particular board was a little difficult, but after some digging, I found Sparkfun maintains [a repo of these files for all of their Arduino based boards](https://github.com/sparkfun/Arduino_Boards). Once I had those, I popped them in the hardware folder and the Arduino IDE happily displayed Serial 7-Segment Display on the list of board options. I also grabbed a copy of the [SevSeg library](https://github.com/sparkfun/SevSeg/tree/master/Libraries/SevSeg) for Arduino, since it's a requirement of their example firmware for the display.

{% highlight c++ %}
// This code uses the SevSeg library, which can be donwnloaded from:
// https://github.com/sparkfun/SevSeg
#include "SevSeg.h" //Library to control generic seven segment displays
SevSeg myDisplay; //Create an instance of the object

#include "Time.h" //Library for keeping track of time
#define TIME_HEADER  "T"   // Header tag for serial time sync message
#define TIME_REQUEST  7    // ASCII bell character requests a time sync message 

//Global variables
unsigned int analogValue6 = 0; //These are used in analog meter mode
unsigned int analogValue7 = 0;
unsigned char deviceMode; // This variable is useds to select which mode the device should be in
unsigned char commandMode = 0;  // Used to indicate if a commandMode byte has been received

time_t t = now();
String minuteString, hourString;
char timeString[5];

void setup()
{ 
  setupDisplay();
  Serial.begin(9600);
  pinMode(13, OUTPUT);
  //setSyncProvider(requestSync);  //set function to call when sync required
  Serial.println("Waiting for sync message");
}

// The display is constantly PWM'd in the loop()
void loop()
{
  if (Serial.available()) {
    processSyncMessage();
  }
  if (timeStatus()!= timeNotSet) {
    digitalClockDisplay();  
  }
  if (timeStatus() == timeSet) {
    digitalWrite(13, HIGH); // LED on if synced
  } else {
    digitalWrite(13, LOW);  // LED off if needs refresh
  }
  //delay(1000);
}

void digitalClockDisplay(){
  
  t = now();
  hourString = String(hourFormat12(t));
  if (hourString.toInt() < 10) { //Because apparently padding in arduino ISN'T A THING.
    hourString = " " + hourString;
  }
  minuteString = String(minute(t));
  if (minuteString.toInt() < 10) {
    minuteString = "0" + minuteString; 
  }
  String(hourString + minuteString).toCharArray(timeString,5);
  // The second argument of DisplayString is for decimal/apostrophe/colon
  // represented as the following bitstring
  //  [MSB] (X)(X)(Apos)(Colon)(Digit 4)(Digit 3)(Digit2)(Digit1)

  myDisplay.DisplayString(timeString, 0b010000); 
  //Serial.println(timeString);
}

void processSyncMessage() {
  unsigned long pctime;
  const unsigned long DEFAULT_TIME = 1357041600; // Jan 1 2013

  if(Serial.find(TIME_HEADER)) {
     pctime = Serial.parseInt();
     if( pctime >= DEFAULT_TIME) { // check the integer is a valid time (greater than Jan 1 2013)
       setTime(pctime); // Sync Arduino clock to the time received on the serial port
       //Serial.println("Got Time " +String(hour(pctime))+":"u+String(minute(pctime)));
     }
  }
}

void setupDisplay(){
 int digit1 = 16; // DIG1 = A2/16 (PC2)
  int digit2 = 17; // DIG2 = A3/17 (PC3)
  int digit3 = 3;  // DIG3 = D3 (PD3)
  int digit4 = 4;  // DIG4 = D4 (PD4)

  //Declare what pins are connected to the segments
  int segA = 8;  // A = D8 (PB0)
  int segB = 14; // B = A0 (PC0)
  int segC = 6;  // C = D6 (PD6), shares a pin with colon cathode
  int segD = A1; // D = A1 (PC1)
  int segE = 23; // E = PB7 (not a standard Arduino pin: Must add PB7 as digital pin 23 to pins_arduino.h)
  int segF = 7;  // F = D7 (PD6), shares a pin with apostrophe cathode
  int segG = 5;  // G = D5 (PD5)
  int segDP= 22; //DP = PB6 (not a standard Arduino pin: Must add PB6 as digital pin 22 to pins_arduino.h)

  int digitColon = 2; // COL-A = D2 (PD2) (anode of colon)
  int segmentColon = 6; // COL-C = D6 (PD6) (cathode of colon), shares a pin with C
  int digitApostrophe = 9; // APOS-A = D9 (PB1) (anode of apostrophe)
  int segmentApostrophe = 7; // APOS-C = D7 (PD7) (cathode of apostrophe), shares a pin with F

  int numberOfDigits = 4; //Do you have a 2 or 4 digit display?

  int displayType = COMMON_ANODE; //SparkFun 10mm height displays are common anode

  //Initialize the SevSeg library with all the pins needed for this type of display
  myDisplay.SetBrightness(100);
  myDisplay.Begin(displayType, numberOfDigits, digit1, digit2, digit3, digit4, digitColon, digitApostrophe, segA, segB, segC, segD, segE, segF, segG, segDP, segmentColon, segmentApostrophe); 
}
{% endhighlight %}

The basic purpose of this example sketch is to set up the seven segment display as a clock synchronized to the programming computer's local time. Perfect for my purposes. After about 10 minutes, it was still keeping good time!

{% include image.html url="/media/2015-09-23-Big-Ole-Watch/FTDIWiring.jpg" width="50%" description="I love the smell of liquid crystal in the....early evening." %} 

All in all, the electronics are pretty much functioning exactly how I want them to. Next up I'll be looking at building the housing for this bad boy and adding some type of interface to set the time, whether that be buttons on the face of the watch, or a port to hook up to a computer for syncing. 
