# SIT210-AlarmStation

// This #include statement was automatically added by the Particle IDE.
#include <LiquidCrystal_I2C_Spark.h>

LiquidCrystal_I2C *lcd;

int lastSecond = 0;
int alarmHour = 9;
int alarmMinute = 35;
int buzzer = A2;
int light = 0;
bool trigger = true;

void setup()
{
  pinMode(buzzer, OUTPUT);
  Serial.begin(9600);
  lcd = new LiquidCrystal_I2C(0x27, 16, 2);
  lcd->init();
  lcd->backlight();
  lcd->clear();
  lcd->print("Sleeping.......");
  Particle.variable("lightSwitch", &light, INT);
  Particle.subscribe("startmovement", StopAlarm);
}

void loop()
{
   CurrentTime();
   if (alarmHour == Time.hour() & alarmMinute == Time.minute())
   {
        lcd->clear();
        light = 1;
        while (trigger == true)
        {
            Particle.publish("startmovement", "alarmBot");
            lcd->setCursor(0,0);
            lcd->print("Goodmorning");
            soundAlarm();
            CurrentTime();
            delay(5000);
        }
   }
}

void StopAlarm(const char *event, const char *data)
{
    if (strcmp(data, "endAlarm") == 0)
    {
        while(trigger == true) 
        {
            lcd->clear();
            lcd->setCursor(0,0);
            lcd->print("Have a great day!");
            CurrentTime();
            alarmHour = -1;
            noTone(buzzer); 
            light = 0;
            delay(1000);
        }
    }
}


void CurrentTime()
{
    if (Time.second() != lastSecond)
    {
    int hour = Time.hour();
    int minute = Time.minute();
    int second = Time.second();
    
    Serial.print(Time.timeStr());
    
    //0 refers too the starting column of the message, and 1 represents the 2nd row
    lcd->setCursor(0,1);
    
    //We are printing time in the correct format of 00:00:00
    lcd->print(hour < 10? "   0" : "    ");
    lcd->print(hour);
    lcd->print(minute < 10? ":0": ":");
    lcd->print(minute);
    lcd->print(second < 10? ":0": ":");
    lcd->print(second);
    lastSecond = Time.second();
    }
}

void soundAlarm()
{
    tone(buzzer, 1000); 
    delay(1000);
    noTone(buzzer); 
    delay(1000);
}
