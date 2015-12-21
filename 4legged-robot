/*
4legged-Robot,4足3D打印机器虫
原创：cnrenhh@gmail.com
2015.12
 */
#include <Wire.h>  //加载I2C库
int mobile_i=0;          //标志，切换到手机控制时,动作组0执行1次
int forword_delay=1300;   //向前延时
int left_delay=1200;     //左转延时
int back_delay=1500;     //后退延时
int right_delay=1100;    //右转延时
int stay_delay=1000;     //原地延时
int r_slide_delay=1800;  //右侧滑延时
int l_slide_delay=1800;  //左侧滑延时
//以下定义前、左、后、右4个避障传感器接在数字口9.10.11.12
int front_sensor = 9;
int left_sensor = 10;
int rear_sensor = 11;
int right_sensor = 12;
//以下定义4个避障传感器返回的值，无障碍返回1，有障碍返回0
int front_state ;
int left_state ;
int rear_state ;
int right_state;
int sensor_status=0;    //4个红外避障传感器整合后的值
#define KEY 2           //D2口为按键中断口
volatile int mode=0;    //中断中使用的变量，模式标志，0-跟随模式(默认)，1-自动模式，2-手机蓝牙控制模式。模式与动作的关系见xls表格

void setup() {
  Wire.begin(); // 本设备加入 i2c 总线，作为主机（主机可以不设置ID号）
  Serial.begin(9600);
  pinMode(13, OUTPUT);
  pinMode(front_sensor, INPUT);
  pinMode(left_sensor, INPUT);
  pinMode(rear_sensor, INPUT);
  pinMode(right_sensor, INPUT);
  attachInterrupt(0,ScanKey,CHANGE); //设置外部中断通道0(引脚2)，引脚电平改变时触发ScanKey函数  
}

void loop() {
float vcc_vol=readVcc()/1000; //将读取的vcc电压mv转换为v  
if (vcc_vol<3.70){             //当vcc电压低于3.7v时，点亮LED灯报警
  digitalWrite(13, HIGH);
  }
Serial.println(vcc_vol,2);
get_sensor();  //获取避障传感器返回值
Serial.println(mode);
switch (mode){      //根据mode值执行不同模式        
  case 0:
     follow_mode();       //跟随模式
     mobile_i=0;          //恢复蓝牙控制模式时，执行一次动作组0
     break;
  case 1:
     auto_mode();        //自动模式
     mobile_i=0;         //恢复蓝牙控制模式时，执行一次动作组0
     break;
  case 2:
     mobile_mode();      //手机蓝牙遥控模式
     break;
  }
delay(500);
}

//读取vcc电压
long readVcc() {
long result;
// Read 1.1V reference against AVcc
ADMUX = _BV(REFS0) | _BV(MUX3) | _BV(MUX2) | _BV(MUX1);
delay(2); // Wait for Vref to settle
ADCSRA |= _BV(ADSC); // Convert
while (bit_is_set(ADCSRA,ADSC));
result = ADCL;
result |= ADCH<<8;
result = 1126400L / result; // Back-calculate AVcc in mV
return result;
}


/*中断函数-start*/
void ScanKey()        //中断发生后，循环改变mode值(0,1,2)    
{
  if(digitalRead(KEY) == LOW)     //有按键按下
  {
    delay(20);            //延时去抖动
    if(digitalRead(KEY) == LOW)   //有按键按下
    {
     switch (mode){
      case 0:
         mode=1;
         break;
      case 1:
         mode=2;
         break;
      case 2:
         mode=0;
         break;
      }
    while(digitalRead(KEY) == LOW); //等待按键松手
    }
  }
}
/*中断函数-end*/

/*自动模式-start*/
void auto_mode(){
  switch (sensor_status){
   case 15:                 //传感器数值15
        action(0x01,forword_delay);       //前进,延时1600ms,时间根据动作完成需要的时间而定
        break;
   case 14:
        action(0x03,left_delay);       //左转,延时1500ms
        break;
   case 13:                      //传感器数值13
        action(0x01,forword_delay);       //前进,延时1600ms
        break;
   case 12:
        action(0x03,left_delay);       //左转,延时1500ms
        action(0x01,forword_delay);       //前进,延时1600ms
        break;
   case 11:
        action(0x04,right_delay);     //右转,延时1500ms
        break;     
   case 10          :            //传感器数值10
        action(0x01,forword_delay);        //前进,延时1600ms
        break;
   case 9:
        action(0x04,right_delay);     //右转,延时1500ms
        action(0x01,forword_delay);     //前进,延时1600ms
        break;   
   case 8:                    //传感器数值8,
        action(0x01,forword_delay);    //前进,延时1600ms
        break;
   case 7:
        action(0x03,left_delay);     //左转并延时6400ms,相当于左转4次，转过90度
        break;   
   case 6:
        action(0x03,left_delay);     //左转,延时1400ms
        action(0x02,back_delay);     //后退，延时1800ms
        break;   
   case 5:
        action(0x09,r_slide_delay);     //右侧滑，延时2000ms
        break;   
   case 4:
        action(0x0a,l_slide_delay);      //左侧滑，延时2000ms
        break;   
   case 3:
        action(0x04,right_delay);     //右转并延时6400ms,相当于右转4次，转过90度
        break;   
   case 2:
        action(0x02,back_delay);     //后退，延时1800ms
        break;   
   case 1:
        action(0x09,r_slide_delay);     //右侧滑，延时2000ms
        break;   
   case 0:
        action(0x00,stay_delay);     //原地不动
        break; 
   default:
        action(0x00,stay_delay);     //原地不动
        break;      
   }
  }
/*自动模式-end*/

/*跟随模式-start*/
void follow_mode(){
  switch (sensor_status){
   case 15:                      //传感器数值15
        action(0x00,stay_delay);       //原地不动,延时1000ms,时间根据动作完成需要的时间而定
        break;
   case 14:
        action(0x04,right_delay);       //右转,延时1400ms
        break;
   case 13:                      //传感器数值13
        action(0x02,back_delay);       //后退,延时1800ms
        break;
   case 12:
        action(0x03,left_delay);       //左转,延时1400ms
        action(0x02,back_delay);       //后退,延时1800ms
        break;
   case 11:
        action(0x03,left_delay);      //左转,延时1400ms
        break;     
   case 10          :            //传感器数值10
        action(0x04,right_delay);        //右转,延时1400ms
        break;
   case 9:
        action(0x04,right_delay);     //右转,延时1400ms
        action(0x02,back_delay);     //后退,延时1800ms
        break;   
   case 8:                    //传感器数值8,
        action(0x02,back_delay);    //后退,延时1800ms
        break;
   case 7:
        action(0x01,forword_delay);     //前进并延时1600ms
        break;   
   case 6:
        action(0x04,right_delay);     //右转,延时1400ms
        action(0x01,forword_delay);     //前进，延时1600ms
        break;   
   case 5:
        action(0x01,forword_delay);     //前进，延时1600ms
        break;   
   case 4:
        action(0x09,r_slide_delay);      //右侧滑，延时2000ms
        break;   
   case 3:
        action(0x03,left_delay);     //左转并延时1400ms
        action(0x01,forword_delay);     //前进
        break;   
   case 2:
        action(0x01,forword_delay);     //前进，延时1600ms
        break;   
   case 1:
        action(0x0a,l_slide_delay);     //左侧滑，延时2000ms
        break;   
   case 0:
        action(0x00,stay_delay);     //原地不动
        break;   
   default:
        action(0x00,stay_delay);     //原地不动
        break;
   }
  }

void mobile_mode(){
  if (mobile_i=0){
      action(0x00,stay_delay);     //原地不动
      mobile_i=1;
  }
}

/*获取避障传感器返回值-start*/
void get_sensor(){
 front_state = digitalRead(front_sensor);
 left_state = digitalRead(left_sensor);
 rear_state = digitalRead(rear_sensor);
 right_state = digitalRead(right_sensor);
 front_state=front_state<<3;
 left_state=left_state<<2;
 rear_state=rear_state<<1;
 sensor_status=front_state+left_state+rear_state+right_state;
// Serial.print("sensor_status:");
// Serial.println(sensor_status);
// Serial.print("state:");
// Serial.print(front_state);
// Serial.print(left_state);
// Serial.print(rear_state);
// Serial.println(right_state);
  }
/*获取避障传感器返回值-end*/

/*向舵机控制板发送动作组数据 */
void action(byte dataL,int delaytime){ 
  Wire.beginTransmission(0x60); //发送数据到舵机控制板，默认I2C地址为0x60
  Wire.write(0x11);      //动作组寄存器地址
  Wire.write(dataL);     //动作组编号低位，一共16个动作组（00~0F）
  Wire.write(byte(0x00));     //动作组编号高位
  Wire.endTransmission();    // 停止发送
  delay(delaytime);
}

