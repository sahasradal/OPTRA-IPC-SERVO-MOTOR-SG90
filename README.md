# OPTRA-IPC-SERVO-MOTOR-SG90
OPTRA IPC ATTEMPT WITH SERVO MOTOR
wiring reminder to my self.
single OG/WH is ground share for arduino& motor
BN = -ve 12v twisted BN/WH
WH = +ve 12v twisted BN/WH

WH/OG WH = -5v
WH/OG OG = +5V

BLUE - D8 - speed signal from IPC
GREEN - A1  - FUEL signal from tank float
WHITE - A0  - temprature signal from engine coolant temprature

######################################################################################################################################################################################################


######################################################################################################################################################################################################



#include <FreqMeasure.h>
#include <ServoTimer2.h>

ServoTimer2 servo1;  // create servo object to control a servo
ServoTimer2 servo2;  // create servo object to control a servo
ServoTimer2 servo3;  // create servo object to control a servo
int val;             // variable
int minval = 2250;                              // 2250 keeps needle at lhs end , 750 keeps the needle /pointer at rhs end
int Speed;  // input for speed is D8
int ThermistorPin = A0;                                                           // thermistor pin declared A1 of aurdino
int fuelPin = A1;                                                                 // Fuel guage float pin A2
int V1;                                                                           // V1 holds the analouge reading of thermistor sensor
float R1 = 10000;                                                                 // variable for temprature calculation
float logR2, R2, T;                                                               // variable for temprature calculation
float c1 = 1.009249522e-03, c2 = 2.378405444e-04, c3 = 2.019202697e-07;           // 
const float offset = 0.50 ;                                                       // offset of sensor
float V2;                                                                         // variable V2 holds fuel sensor reading
int C ;            // calculated quantity ,fuel tank 
double AVGC;       // variable to hold 100 cumulative readings of fuel level  // accumulator for 100 readings
double AVGT;       // variable to hold 100 cumulative readings of temprature  // accumulator for 100 readings
int a=0;           // fuel guage read count
int CC;            // will hold the final averaged fuel level
int TT;            // will hold the final averaged temprature 
double count;
int frequency;
void setup() {
  Serial.begin(9600);
  FreqMeasure.begin();
  servo1.attach(3);  //  speedometer
  servo2.attach(6);  //  temprature guge
  servo3.attach(9);  //  fuel guage
  pinMode(A0, INPUT);
  pinMode(A1, INPUT);
  servo1.write(minval);  // pulse width for 0 speeed (reset guage at start)
  delay(500); 
  servo2.write(minval);  //pulse width for 0 degree temprature (reset guage at start)
  delay(500); 
  servo3.write(minval);  //pulse width for 0liter fuel (reset guage at start)
  delay(500); 
  
}




void loop() {
  
  if (FreqMeasure.available()) {
    // digital pin D8 for frequency read
    double count =  FreqMeasure.read();
    //count = FreqMeasure.countToFrequency(count);  // using this cause 30 sec delay
    frequency = 15998286/count;                     // 15998286 is the count generated by FreqMeasure with 50% 500ms cycle
    //Speed = (minval-((count /1.81)*10)) ;
    Speed = (minval-((frequency /1.81)*10)) ;
    if (Speed > 2250){ Speed = 2250;}
    if (Speed < 750) { Speed = 750;}
}
// 0kmph = 2250, 150kmph = 750 , 1500/150 = 10 per kmph              // motor has 750 at one end and 2250 at other end, we have 2250 at left side so subtract reading from 2250 to get sweep frpm left to right

  V1 = analogRead(ThermistorPin);                                    // read and store thermistor value in V1
   T = (minval-((((1024-V1)/10)+33)*10)); // calculate and store V1 in T
    if (T > 2250){ T = 2250;}
    if (T < 750) { T = 750;}
// 0deg =2250 , 150 deg =750 , 1500/150 = 10 per degree              // motor has 750 at one end and 2250 at other end, we have 2250 at left side so subtract reading from 2250 to get sweep frpm left to right  
   V2 = analogRead(fuelPin);                                         // read and store fuels sensor value in V2
   C = (minval-(((400-V2)/5.4)*27)); 
    if (C > 2250){ C = 2250;}
    if (C < 750) { C = 750;}
// 0L=2250 , 55L=750 , 2250-750 =1500 , 1500/55 = 27 per liter       // motor has 750 at one end and 2250 at other end, we have 2250 at left side so subtract reading from 2250 to get sweep frpm left to right
   
  servo1.write(Speed);  //min pulse width for 0 degree 
  delay(5); 
  servo2.write(T);  //pulse width for 90 degree 
  delay(5); 
  servo3.write(CC);  //max pulse width for around 180 degree 
  delay(5);
   count = 0;
  //Serial.println(Speed);
  //Serial.println(T);
  //Serial.println(CC);
  
  a+=1;                           // increase variable a everytime the loop reaches here
  AVGC += C;                      // add the fuel reading to variable AVGC each iteration
  if (a >= 60){C= AVGC/60;        // if variable a is 100 or above calculate the average of 100 fuel readings to avoide fuel slosh/erratic read & store in C
  a=0;                            // clear a
  AVGC = 0;  
  CC = C;
  C = 0;
  }
}

