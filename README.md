#define PIN_LED   9
#define PIN_TRIG 12
#define PIN_ECHO 13


#define SND_VEL      346.0
#define INTERVAL       25     
#define PULSE_DURATION 10      
#define _DIST_MIN    100.0   
#define _DIST_MAX    300.0     
#define TIMEOUT ((INTERVAL / 2) * 1000.0)
#define SCALE   (0.001 * 0.5 * SND_VEL)  

unsigned long last_sampling_time = 0;

void setup() {
  pinMode(PIN_LED, OUTPUT);
  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  digitalWrite(PIN_TRIG, LOW);

  Serial.begin(57600);
}

void loop() {
  if (millis() - last_sampling_time < INTERVAL) return;
  last_sampling_time = millis();

  float distance = USS_measure(PIN_TRIG, PIN_ECHO);

  int duty; 

  if (distance == 0.0 || distance <= 0) {
    duty = 255; 
  } else if (distance <= _DIST_MIN) {      
    duty = 255;
  } else if (distance < 200.0) {         
    duty = (int)((200.0 - distance) * 255.0 / 100.0 + 0.5);
  } else if (distance == 200.0) {            
    duty = 0;
  } else if (distance <= _DIST_MAX) {           
    duty = (int)((distance - 200.0) * 255.0 / 100.0 + 0.5);
  } else {                                     
    duty = 255;
  }



  analogWrite(PIN_LED, constrain(duty, 0, 255));

  Serial.print("Min:"); Serial.print(_DIST_MIN);
  Serial.print(", distance:"); Serial.print(distance);
  Serial.print(", Max:"); Serial.print(_DIST_MAX);
  Serial.print(", duty:"); Serial.println(duty);
}


float USS_measure(int TRIG, int ECHO) {
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(PULSE_DURATION);
  digitalWrite(TRIG, LOW);
  return pulseIn(ECHO, HIGH, TIMEOUT) * SCALE; 
}
