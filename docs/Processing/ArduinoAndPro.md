# Arduino & Processing

Task: Use arduino's rocker to control processing's "spider" to walk.

### 1  Learn how to use a rocker
<img src="https://github.com/Fy1307/IMGofSixGod/blob/master/img/Frame_10.jpg?raw=true" width = "1000" div align= 'center' /><br><br/>

```jsx
#define pinX  A0  //Declare port
#define pinY  A1
int value1 = 0;
int value2 = 0;
int x;

void setup()
{
  pinMode(pinX, INPUT); //Declare ports set A0 and A1 as inputs
  pinMode(pinY, INPUT);
  Serial.begin(9600);  //Set the baud rate on both ends of the same baud rate can communicate
}

void loop()
{
  value1 = analogRead(pinX); 
  value2 = analogRead(pinY);
  Serial.print(value1);  //Read the values uploaded on the X-axis
  Serial.print(",");
  Serial.println(value2); //Read the values uploaded on the Y-axis
  delay(100);
}
```

Depending on the above code, you can read such data in the serial port monitor.

<img src="https://github.com/Fy1307/IMGofSixGod/blob/master/img/data_arduino.png?raw=true" width = "1000" div align= 'center' /><br><br/>

By manipulating the rocker from one end to the other, the x or y axis output ranges from 0 to 1023. When the stick is in the neutral position, it outputs a median value of 500+, as shown above.


### 2  Let processing read the data from the arduino 

The key code is as follows, with which the two parties can communicate:

```jsx
import processing.serial.*;
Serial myPort;         // Set up a port
int input;              // Declare a variable assigned to the data it receives from the arduino

void setup() {
    port = new Serial(this,"COM3",9600); //Set up a channel and a rate that must be consistent with the Arduino
}

void draw() {
   int input = port.read();  //The serial port receives data to the input
}
```

```jsx

void setup()
{
  Serial.begin(9600);  //Set the baud rate, the same baud rate on both ends can communicate
}

void loop()
{
	Serial.write();  //Upload data from the arduino to processing
  delay(100);
}
```

### 3  Let the spider follow the rocker

At first, my idea is to assign the X-axis data to the variable x in processing, the Y-axis data to the variable y in processing, and then use a map mapping function to limit the value of x and y (originally should be 0-1023) to the length and width of the canvas, and then use the variables x and y to control the position of the spider.

<img src="https://github.com/Fy1307/IMGofSixGod/blob/master/img/Arduino3.png?raw=true" width = "1000" div align= 'center' /><br><br/>

However, the problem with this method is that arduino will continuously output the values of x and y, which processing cannot distinguish, as shown in the figure above. (Of course, there must be a way to solve it, but I have a weak foundation T T, there is no good way) So the last method used is: first, read the value of the rocker in arduino, set 8 states (corresponding to 8 moving ways), then read the state by processing, and slowly change the value of x and y.

### 4  Code

```jsx
/*For Processing*/
import processing.serial.*;
Serial port;
int v =1;
int STEP = 40; //??????????????????
int x0 = 0, y0= 0;
float vx = 0, vy = 0; //????????????
int StepRound(int i) { //??????????????????????????????
    return int(round(float(i) / STEP) * STEP);
}

void setup() {
    size(800, 800);//????????????
    stroke(255);//????????????
    port = new Serial(this,"COM3",9600);
}

int dist(int x1, int y1,int x2, int y2) {//????????????dist()
    return int(dist(float(x1),float(y1),float(x2),float(y2)));
}

void draw() {
    background(0); //????????????
    int input = port.read(); //??????arduino?????????
    if(input == 1){  //?????????????????????
      vx+=v;  //??????x???????????????
      vy+=v;  //??????y???????????????
    }
    if(input == 2){ //????????????????????????????????????
      vx+=v;
      vy-=v;}    
    if(input == 3){
      vx-=v;
      vy+=v;}
    if(input == 4){
      vx-=v;
      vy-=v;}
     if(input == 5) //??????????????????
      vx+=v;
    if(input == 6) //?????????????????????????????????
      vx-=v;
    if(input == 7) 
      vy-=v;
    if(input == 8) 
      vy+=v;
    if(input == -1){  //???????????????????????????????????????????????????????????????-1???????????????????????????????????????????????????
		vx*=0.95; 
    vy*=0.95;}
    
    x0+=round(vx); //????????????????????????round????????????????????????????????????????????????????????????????????????
    y0+=round(vy);
      
    if(x0<20) x0=20;  //???????????????????????????????????????????????????
    if(x0>780) x0=780;
    if(y0<20) y0=20;
    if(y0>780) y0=780;
    ellipse(x0, y0,STEP / 4 ,STEP / 4);//????????????
    for (int i = StepRound(x0) - STEP * 2; i <= StepRound(x0) + STEP * 2; i += STEP) {//????????????
        for (int j = StepRound(y0) - STEP * 2; j <= StepRound(y0) + STEP * 2; j += STEP) {
            int leg = dist(x0, y0, i, j);//??????????????????
            ellipse(i, j,(STEP * 3 - leg) / STEP + 1,(STEP * 3 - leg) / STEP + 1);
            if (leg > STEP && leg < STEP * 2) //??????????????????
                line(x0, y0, i, j);//?????????
        }
    }
    //delay(10);//*****
}
```

```jsx
/*For Arduino*/
#define pinX  A0  //????????????
#define pinY  A1
#define left 480
#define right 560
int value1 = 0;
int value2 = 0;

void setup()
{
  pinMode(pinX, INPUT); //??????A0 A1?????????????????????
  pinMode(pinY, INPUT);
  Serial.begin(9600);  //??????????????? ?????????????????????????????????
}

void loop()
{
  value1 = analogRead(pinX); 
  value2 = analogRead(pinY);
  if(value1 > right && value2 > right)
  Serial.write(1);
  else if(value1 > right && value2 <left)
  Serial.write(2);
  else if(value1 < left && value2 >right)
  Serial.write(3);
  else if(value1 < left && value2 <left)
  Serial.write(4);
  else if(value1 >= right && value2 >=left && value2 <=right)
  Serial.write(5);
  else if(value1 < left && value2 >left && value2 <right)
  Serial.write(6);
  else if(value2 < left && value1 >left && value1 <right)
  Serial.write(7);
  else if(value2 > right && value1 >left && value1 <right)
  Serial.write(8);
  delay(50);
}

```

### 5  Final result
<img src="https://github.com/Fy1307/IMGofSixGod/blob/master/img/Arduino_Result.GIF?raw=true" width = "1000" div align= 'center' /><br><br/>


### 6  Problems & Solutions
- problem description
    
    When using a Mac computer to connect to Arduino, you need to use the docking station. When running, you need to change and confirm the "port", but the processing code will report an error after entering and running. When it's time to switch to a Win computer, there's no problem, and you can run and interact normally.
    
    The specific error content is as follows:
    
    ```cpp
    A library used by this sketch relies on native code that is not available.
    UnsatisfiedLinkError: Could not load the jssc library: Couldn't load library library jssc
    Could not run the sketch (Target VM failed to initialize).
    For more information, read Help ??? Troubleshooting.
    ```
    <div align= 'left'>
    <img src="https://github.com/Fy1307/IMGofSixGod/blob/master/img/PAPro1.png?raw=true" width = "1000" />
    </div>
    <br></br>
    
- Solution???
    <div align= 'left'>
    <img src="https://github.com/Fy1307/IMGofSixGod/blob/master/img/PAPro2.png?raw=true" width = "1000" />
    </div>
    <br></br>
    
    - Step 1: Click the URL link [https://github.com/java-native/jssc/releases](https://github.com/java-native/jssc/releases)
    - Step 2: Download the file [jssc-2.9.4.jar](https://github.com/java-native/jssc/releases/download/v2.9.4/jssc-2.9.4.jar)
    - Step 3: Go to the folder "Applications" - find processing - right click "Show Package Contents"
    
    <div align= 'left'>
    <img src="https://github.com/Fy1307/IMGofSixGod/blob/master/img/PAPro3.png?raw=true" width = "1000" />
    </div>
    <br></br>
    
    - Step 4: Follow the following path /Contents/Java/modes/java/libraries/serial/library/ and find the original jssc.jar and replace it with the latest version jssc-2.9.4.jar that you just downloaded
    - Step 5: Restart processing, you can solve the problem
  
- Reference
    
    [https://github.com/java-native/jssc/releases](https://github.com/java-native/jssc/releases)  