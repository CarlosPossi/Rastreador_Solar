## ![c-](https://github.com/user-attachments/assets/a2c114fb-12e2-48be-afca-b794e443442e) Programação do Protótipo: 
```
# include <Servo.h>

#define estaDia true
#define estadoNoite false

int pinServoVert = 9;
int pinServoHoriz = 8;
int chave = 2;

int pinSensorSupEsq = A0;
int pinSensorSupDir = A2;
int pinSensorInfEsq = A1;
int pinSensorInfDir = A3;

Servo servoVert;
Servo servoHoriz;

bool estadoDia = estadoNoite;

int posinicial = 90;

int anguloVert = posinicial;
int anguloHoriz = posinicial;
int valorNoite = 100;

void setup ()
{
pinMode (chave, INPUT_PULLUP);

pinMode (pinSensorSupEsq, INPUT);
pinMode (pinSensorSupDir, INPUT);
pinMode (pinSensorInfEsq, INPUT);
pinMode (pinSensorInfDir, INPUT);
  
servoVert.attach (pinServoVert);
servoHoriz.attach (pinServoHoriz);
  
Serial.begin (9600);
}

void loop ()
{

estadoDia = !estaNoite();

if (!estaManual()) 
{
if (estadoDia)
{
 servoVert.write (constrain (anguloVert, 0, 180));
 servoHoriz.write(constrain (anguloHoriz, 0, 180));
  
 anguloHoriz += comparaHoriz();
 anguloVert += comparaVert();
}
  
else
{
servoVert.write (posinicial);
servoHoriz.write(posinicial);
}

delay (20);
}
}


int comparaHoriz()
{
int valorEsq = analogRead (pinSensorSupEsq) + analogRead (pinSensorInfEsq);
int valorDir = analogRead (pinSensorSupDir) + analogRead (pinSensorInfDir);
  
if (valorEsq > valorDir)
{
return 1;
}
  
else if (valorEsq < valorDir)
{
return -1;
}
  
else
{
return 0;
}
}

int comparaVert()
{
int valorSup = analogRead (pinSensorSupEsq) + analogRead (pinSensorSupDir);
int valorInf = analogRead (pinSensorInfEsq) + analogRead (pinSensorInfDir);
  
if (valorSup > valorInf)
{
return 1;
}
  
else if (valorSup < valorInf)
{
return -1;
}
  
else
{
return 0;
}
}

bool estaNoite()
{
return ((analogRead (pinSensorSupEsq) < valorNoite) && (analogRead (pinSensorSupDir) < valorNoite) && (analogRead (pinSensorInfEsq) < valorNoite) && (analogRead (pinSensorInfDir) < valorNoite));
}

bool estaManual() 
{
return digitalRead(chave);
}
```
<br>
<br>
  
## ![vision](https://github.com/user-attachments/assets/52ec7d96-c5ed-49df-b477-f17b858a7fdb) Programação da Visão Computacional:
```

```
