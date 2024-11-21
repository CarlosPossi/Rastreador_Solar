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
import cv2
import numpy as np
import RPi.GPIO as GPIO
import time

# Função para inicializar o sistema de servo
def setup_servo(servo_pin):
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(servo_pin, GPIO.OUT)
    pwm = GPIO.PWM(servo_pin, 50)  # Frequência de 50 Hz
    pwm.start(0)
    return pwm

# Função para mover o servo com base na posição calculada do sol
def move_servo(pwm, position):
    # Converte a posição do sol para um valor de ângulo para o servo
    angle = (position / 360.0) * 180  # Mapeando para 0-180 graus
    duty_cycle = (angle / 18) + 2
    pwm.ChangeDutyCycle(duty_cycle)
    time.sleep(1)

# Função principal que controla a captura de imagem, processamento e movimentação do painel solar
def rastrear_sol():
    # Inicializa a câmera
    cap = cv2.VideoCapture(0)  # Usando a câmera conectada
    
    # Inicializa o controle do servo (ajuste de azimute, por exemplo)
    servo_pin = 17  # Pino do servo conectado no Raspberry Pi
    pwm = setup_servo(servo_pin)

    while True:
        ret, frame = cap.read()
        if not ret:
            print("Erro ao capturar imagem")
            break

        # Converter a imagem para o espaço de cores HSV
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        # Definir o intervalo de cor do sol (amarelo)
        lower_yellow = np.array([20, 100, 100])
        upper_yellow = np.array([40, 255, 255])

        # Criar uma máscara para isolar a cor amarela (do sol)
        mask = cv2.inRange(hsv, lower_yellow, upper_yellow)

        # Encontrar contornos que podem representar o sol
        contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        if contours:
            largest_contour = max(contours, key=cv2.contourArea)
            (x, y), radius = cv2.minEnclosingCircle(largest_contour)

            # Desenhar o contorno do sol na imagem
            cv2.circle(frame, (int(x), int(y)), int(radius), (0, 255, 0), 2)
            cv2.putText(frame, "Sol Detectado", (int(x), int(y) - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)

            # Calcular a posição do sol em termos de ângulo de azimute
            # Aqui, apenas uma simulação simples. Use a coordenada x da imagem para determinar o azimute.
            azimute = (x / frame.shape[1]) * 360  # Normaliza o valor de x para 0 a 360 graus

            # Controlar o servo para ajustar a posição do painel solar
            move_servo(pwm, azimute)

        # Mostrar a imagem com a posição do sol detectada
        cv2.imshow('Posição do Sol', frame)

        # Sair do loop pressionando 'q'
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()
    pwm.stop()
    GPIO.cleanup()

# Executa o rastreamento do sol
if __name__ == "__main__":
    rastrear_sol()

```
<h2 align="center"> Descrição do Código:</h2>

#### 🔹 Captura de Imagem com a Câmera:

A câmera é inicializada usando a biblioteca cv2.VideoCapture(0). Ela captura uma imagem do céu a cada iteração no loop principal.
<br>
#### 🔹Processamento de Imagem para Detectar o Sol:
A imagem capturada é convertida para o espaço de cores HSV com cv2.cvtColor(frame, cv2.COLOR_BGR2HSV). Em seguida, a segmentação de cor é feita com o intervalo de cores correspondente ao amarelo ou branco, que são características comuns do sol.
Utilizamos a função cv2.inRange para criar uma máscara que isola as partes da imagem com a cor do sol.
O algoritmo findContours detecta os contornos, e o maior contorno é assumido como o sol.
<br>
#### 🔹Cálculo da Posição do Sol:
A posição do sol é calculada a partir das coordenadas x do centro do círculo detectado. Isso é usado como uma aproximação para o azimute do sol.
O valor de x da imagem é mapeado para um ângulo de 0 a 360 graus (azimute), com base na largura da imagem.
<br>
#### 🔹Controle do Servo para Ajustar o Painel Solar:
O valor de azimute é utilizado para controlar um servo motor que ajusta a posição do painel solar. O valor do azimute mapeado para um ângulo de servo (0 a 180 graus) é convertido para um ciclo de trabalho de PWM (Pulse Width Modulation), que é enviado ao servo para movimentá-lo.
<br>
#### 🔹Exibição de Imagens e Controle do Sistema:
A imagem com o sol detectado é exibida em tempo real com a função cv2.imshow().
O código roda em um loop até que a tecla 'q' seja pressionada.
<br>
#### 🔹Encerramento e Limpeza:
Ao final, a câmera é liberada e a janela de exibição é fechada com cv2.destroyAllWindows().
O PWM do servo é parado e o GPIO é limpo com GPIO.cleanup().


