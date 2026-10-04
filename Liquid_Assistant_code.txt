#include <OneWire.h>
#include <DallasTemperature.h>

#define ONE_WIRE_BUS 2
#define Red 3
#define Green 4
#define Blue 5
#define level A0
#define buzzer 13

#define buzzer_start 400  //수위센서 작동수위: 400

#define bluetemp 20      //파란불: 섭씨 20도 이하
#define redtemp 36       //빨간불: 섭씨 36도 이상

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);
int temperature;              //온도를 저장할 변수
double water;                 //수위를 저장할 변수
unsigned long lastactivate=0;  //가장 최근에 부저가 작동한 시간

int tempsensor() //온도계 측정 함수
{
    sensors.requestTemperatures(); // 온도를 입력받기 위한 명령어
    return (int)sensors.getTempCByIndex(0); //온도(섭씨) 반환
}

void levelbuzzer() //부저 작동 함수
{
    if(water<buzzer_start) //수위가 지정 수위보다 낮으면
        return;
    else if(millis()-lastactivate>20000 || lastactivate==0) //수위가 지정 수위보다 높고 부저가 작동 한 지 20초가 지났으면
    {
        lastactivate=millis(); //부저가 작동한 시간을 저장
        digitalWrite(buzzer,HIGH); //(1초 작동 + 0.5초 정지)*3회
        delay(1000);
        digitalWrite(buzzer,LOW);
        delay(500);
        digitalWrite(buzzer,HIGH);
        delay(1000);
        digitalWrite(buzzer,LOW);
        delay(500);
        digitalWrite(buzzer,HIGH);
        delay(1000);
        digitalWrite(buzzer,LOW);
        delay(500);
        return;
    }

void LED(int temp) //LED 작동 함수
{
    if(temp<bluetemp) //파란색 점등
    {
        digitalWrite(Red,LOW);
        digitalWrite(Green,LOW);
        digitalWrite(Blue,HIGH);
    }
    else if(temp>redtemp) //빨간색 점등
    {
        digitalWrite(Red,HIGH);
        digitalWrite(Green,LOW);
        digitalWrite(Blue,LOW);
    }
    else //노란색 점등
    {
        digitalWrite(Red,HIGH);
        digitalWrite(Green,HIGH);
        digitalWrite(Blue,LOW);
    }
}

void setup(void)
{
    sensors.begin(); //센서 통신 시작
    pinMode(Red,OUTPUT);
    pinMode(Green,OUTPUT);
    pinMode(Blue,OUTPUT);
    pinMode(buzzer,OUTPUT);

    digitalWrite(Red,LOW);
    digitalWrite(Green,LOW);
    digitalWrite(Blue,LOW);
}

void loop(void)
{
    temperature=tempsensor(); //수온 측정&저장
    water=analogRead(level); //수위 측정&저장
    LED(temperature); //LED 함수 호출
    levelbuzzer(); //부저작동 함수 호출
}
