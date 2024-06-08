![This is an alt text.](/image/leanbot.jpg "Image of Leanbot.")


## Problem Statement
[Leanbot](https://leanbot.space/), a Arduino based robotkit from Vietnam, is used in competition to complete various tasks to win the competition. Different competition required different task so Leanbot code need to change to match the tasks. This is a very time consuming process. Measuring the Leanbot distance and orientation then modify a small section of the code for every iteration is a tideous trial and error process. 

## Solution
Rather than doing the measurement and modify code instruction manually, bluetooth communication can be added into leanbot for data collection and send command to perform task then record the right sequences of tasks to streamline the process. As a result of this features, the trial and error process is **shorten from 2 weeks to 2 days**. 

## Readme structure
- [Leanbot Platform](#leanbot-platform)
  - [Built-in Function](#built-in-function)
- [Arduino Code](#arduino-code)
  - [Bluetooth communication](#bluetooth-communication)
  - [Data Parsing](#data-Parsing)
  - [Record and Play](#record-and-play)
- [Limitation](#limitation)

## Leanbot Platform
Leanbot come with many function out of the box. It include stepper motor control, gripper, ultrasonic sensor, RGB Led, MotionTracking sensor, IR sensor array, buzzer, bluetooth module and more. Since its microcontroller is arduino, all of the electronic components are open source as well. Many resources are available to tinker the Arduino ecosystem. What make Leanbot unique is that its arduino come in ready to use along with beginner-friendly programming platform to start children age between 7 to 15 their learning journey in STEM. 

### Built-in function
To program leanbot and utilise their custom made library function, go to their [online IDE website](https://ide.pythaverse.space/). 


## Arduino Code
Here is the arduino code of the controller:
```cpp
#include <Leanbot.h> 

//raw data from serial monitor
const byte numChars = 20;
char receivedChars[numChars];
char tempChars[numChars];  // temporary array for use when parsing

boolean newData = false;

// variables to hold the parsed data

struct messageFromPC {
  char message[numChars];
  int input[4];
};

//variable to follow line
int target, speed, line, distance, test;
bool followlineOn;

// buffer for record and play
const int buffer_size = 15;
static int readPosition = 0;
static int writePosition = 0;
char buffer[buffer_size][numChars]={0};

bool record = false;

//============ 

void setup() { 

  Leanbot.begin();                    // initialize Leanbot 

														 
	  
  //LbIRLine.setThreshold(202, 176, 174, 167);          
											  
  //for irLine following because the ir sensor don't know what is black or white. it is relative 
  Serial.println("Welcome to Leanbot Bluetooth remote control."); 
  Serial.println("Enter function below with data in this style HelloWorld(12, 24.7)  "); 
  Serial.println("1. Movemm(speed, distance) - Forward and/or Backward with max speed 55"); 
  Serial.println("2. Movems(speed, time) - Forward and/or Backward with max speed 55"); 
  Serial.println("3. Turndeg(left, right, degree) - Point and/or Swing"); 
  Serial.println("4. Turnms(left, right, time) - Point and/or Swing"); 
  Serial.println("5. follow(speed, distance) - follow line with speed and distance"); 
  Serial.println("6. gripon/gripoff - Gripper Turn on/off"); 
  Serial.println("7. Use Record to write comamnd into buffer then Done to go back testing.");
  Serial.println("8. Use Play to read from buffer.");  
  Serial.println("input[0] , input[1] , input[2] , input[3] ");

} 

void loop(){
  recvWithEndMarkers();

  if (newData == true) {
    strcpy(tempChars, receivedChars);
    // this temporary copy is necessary to protect the original data
    // because strtok() used in parseData() replaces the commas with \0
    if (record == true) {
      if (strcmp(tempChars, "done") == 0) {
        Serial.println("Record done.");
        record = false;
        strcpy(tempChars, " ");       //clear message
      }
      else{
        // record the buffer
        strcpy(buffer[writePosition], tempChars);
        int i = writePosition;
        i++;
        Serial.print(i); Serial.print(". ");
        Serial.print(buffer[writePosition]); 
        Serial.print(" & "); Serial.println(tempChars);
        writePosition++;
        strcpy(tempChars, " ");       //clear message
      }
    }

    messageFromPC command = parseData(tempChars);
    messageFromPC *cptr = &command;
    dataCommand(cptr);
    //showParsedData(cptr);

    newData = false;
  }
}

  
//============ 

 void recvWithEndMarkers() { 
    static byte ndx = 0; 
    char endMarker = '\n'; 
    char rc; 
     while (Serial.available() > 0 && newData == false) { 
      rc = Serial.read(); 
          if (rc != endMarker) { 
              receivedChars[ndx] = rc; 
              ndx++; 
              if (ndx >= numChars) { 
                  ndx = numChars - 1; 
              } 
          } 
          else { 
              receivedChars[ndx] = '\0'; // terminate the string 
              ndx = 0; 
              newData = true; 
          } 
      } 
} 

 

//============ 

struct messageFromPC parseData(char data[numChars]) {  // split the data into its parts
  char *strtokIndx;  // this is used by strtok() as an index
  messageFromPC m;
  strtokIndx = strtok(data, "(");  // get the first part - the string
  strcpy(m.message, strtokIndx);  // copy it to messageFromPC

  strtokIndx = strtok(NULL, ",");  // this continues where the previous call left off
  m.input[0] = atoi(strtokIndx);

  strtokIndx = strtok(NULL, ",");  // this continues where the previous call left off
  m.input[1] = atoi(strtokIndx);

  strtokIndx = strtok(NULL, ",");
  m.input[2] = atoi(strtokIndx);

  strtokIndx = strtok(NULL, ")");  // this continues where the previous call left off
  m.input[3] = atoi(strtokIndx);

  return m;
}

 

//============ 

 

void dataCommand(messageFromPC *m){ 
  if(strcmp(m->message, "movemm") == 0){ 
    Serial.print("movemm called "); 
    Serial.print("speed= ");
    Serial.print(m->input[0]);
    Serial.print(" distance in mm= ");
    Serial.println(m->input[1]);
    if(m->input[0]>0 || m->input[1]>0 ){ 
      //showParsedData(); 
      LbMotion.runLRrpm(m->input[0], m->input[0]);
      LbMotion.waitDistanceMm(m->input[1]);
      LbMotion.stopAndWait();
    } 
  } 
  else if(strcmp(m->message, "movems") == 0){ 
    Serial.print("movetime called "); 
    Serial.print("speed= ");
    Serial.print(m->input[0]);
    Serial.print(" time in ms= ");
    Serial.println(m->input[1]);
    if(m->input[0]>0 || m->input[1]>0 ){ 
      //showParsedData(); 
      LbMotion.runLRrpm(m->input[0], m->input[0]);
      LbDelay(m->input[1]);
      LbMotion.stopAndWait();
    } 
  } 
  else if(strcmp(m->message, "turndeg") == 0){ 
    Serial.print("Turndeg called "); 
    Serial.print("left= ");
    Serial.print(m->input[0]);
    Serial.print(" right= ");
    Serial.print(m->input[1]);
    Serial.print(" degree= ");
    Serial.println(m->input[2]);
    if(m->input[0]>0 || m->input[1]>0 || m->input[2]> 0 ){ 
      //showParsedData(); 
       LbMotion.runLRrpm(m->input[0], m->input[1]);
       LbMotion.waitRotationDeg(m->input[2]);
       LbMotion.stopAndWait();
    } 
  } 
    else if(strcmp(m->message, "turnms") == 0){ 
    Serial.print("Turnms called "); 
    Serial.print("left= ");
    Serial.print(m->input[0]);
    Serial.print(" right= ");
    Serial.print(m->input[1]);
    Serial.print(" time in ms= ");
    Serial.println(m->input[2]);
    if(m->input[0]>0 || m->input[1]>0 || m->input[2]> 0 ){ 
      //showParsedData(); 
       LbMotion.runLRrpm(m->input[0], m->input[1]);
       LbDelay(m->input[2]);
       LbMotion.stopAndWait();
    } 
  } 
    else if(strcmp(m->message, "gripon") == 0){ 
    Serial.println("Grip On called"); 
    LbGripper.close();
  } 
    else if(strcmp(m->message, "gripoff") == 0){ 
    Serial.println("Grip Off called"); 
    LbGripper.open();
  }
    else if (strcmp(m->message, "record") == 0) {
    Serial.println("Record called. You may write to buffer now. ");
    record = true;
  }
    else if (strcmp(m->message, "show") == 0) {
    Serial.println("Here's the current buffer:");
    int x = 0;
    for(int i = 0 ; i < writePosition ; i++){
      x++;
      Serial.print(x); Serial.print(". ");
      Serial.println(buffer[i]);
    }
  }
    else if (strcmp(m->message, "play") == 0) {
    Serial.println("Play called ");
    play();
  }
  else if (strcmp(m->message, "replay") == 0) {
    Serial.println("Replay called ");
    replay();
  }
    else if (strcmp(m->message, " ") == 0) {
    //do nothing......
  }
    else if(strcmp(m->message, "follow") == 0){ 
    Serial.println("Follow line called"); 
    Serial.print("speed= ");
    Serial.print(m->input[0]);
    Serial.print(" distance= ");
    Serial.println(m->input[1]);
    if(m->input[0]>0 || m->input[1]>0 ){ 
      //showParsedData(); 
      speed = m->input[0]; 
      distance = m->input[1]; 
      linefollowing();
    } 
  } 
    else if(strcmp(m->message, "followlineoff") == 0){ 
    Serial.println("Follow line Off called"); 
    linefollowingOff();
  }  
  else if(strcmp(m->message, "receivedData") == 0){
    Serial.print("Ultrasonic: "); 
    Serial.println(Leanbot.pingCm());   
    Serial.print("degree: "); 
    Serial.println(LbMotion.getRotationDeg());  
    Serial.print("distance: "); 
    Serial.println(LbMotion.getDistanceMm() );
  }
  else if(strcmp(m->message, "reset") == 0){
    Serial.println("Reset Done");      
    LbMotion.setZeroOrigin();
    writePosition = 0;
    readPosition = 0;
    memset(buffer, 0, sizeof(buffer[0][0]) * buffer_size * numChars);
  }
  else{ 
    Serial.println("Invalid command"); 
  } 
} 
  

//========================

void showParsedData(messageFromPC *m) {
  Serial.print("Message: ");
  Serial.println(m->message);
  Serial.print("Input[0]: ");
  Serial.println(m->input[0]);
  Serial.print("Input[1]: ");
  Serial.println(m->input[1]);
  Serial.print("Input[2]: ");
  Serial.println(m->input[2]);
  Serial.print("Input[3]: ");
  Serial.println(m->input[3]);
}

//==============================

void play(){
    static int x = 0;
    while(readPosition < writePosition) {
      x++;
      Serial.print(x); Serial.print(". ");
      Serial.println(buffer[readPosition]);
      messageFromPC command = parseData(buffer[readPosition]);
      messageFromPC *cptr = &command;
      dataCommand(cptr);
      readPosition++;
    }

    Serial.println("End of buffer.");
}
//==============================

void replay(){
    int x = 0;
    Serial.println("Replay buffer.");
    while(x < writePosition) {
      int i = 0;
      i++;
      Serial.print(x); Serial.print(". ");
      Serial.println(buffer[x]);
      messageFromPC command = parseData(buffer[x]);
      messageFromPC *cptr = &command;
      dataCommand(cptr);
      x++;
    }

    Serial.println("End of buffer.");
}

/******************************************************************************* 

Leanbot_LineFollowing 

*******************************************************************************/ 

void linefollowing() { 

  followlineOn = true; 
  followLine();                      // light status update 

} 

  

void linefollowingOff() { 

  followlineOn = false;                       // light status update 

} 

  
void followLine() {
  target = LbMotion.getDistanceMm() + distance;
  followLineCarefully();
  while (LbIRLine.isBlackDetected()) {
    followLineCarefully();
    if ((LbMotion.getDistanceMm() > target)) {
      if ((LbIRLine.value() == 6)) {
        followlineOn = false;
        break;
      }
    }
    if (!followlineOn) {
      break;
    }
    LbMotion.waitDistanceMm(1);
  }
  LbMotion.stopAndWait();
  LbDelay(500);
}

void followLineCarefully() {
  for (int count = 0; count < 10; count++) {
    runFollowLine();
    if (LbIRLine.isBlackDetected()) {
      break;
    }
    LbMotion.waitDistanceMm(1);
  }

}

void runFollowLine() {
  LbIRLine.read();
  line = LbIRLine.value();
  LbIRLine.displayOnRGB(0x00ff00);
  switch (line) {
    case 4:
    case 14:
    case 12:
    case 8:
      LbMotion.runLRrpm(speed / 8, speed);
      break;

    case 2:
    case 7:
    case 3:
    case 1:
      LbMotion.runLRrpm(speed, speed / 8);
      break;

    case 9:
    case 13:
    case 11:
    case 10:
    case 5:
      test = 123;
      break;

    default:
      LbMotion.runLRrpm(speed, speed);
   }

}
```

###Bluetooth communication
  
  
###Data Parsing
 
 
###Record and Play
  
  
  
  
  
  
##Limitation









