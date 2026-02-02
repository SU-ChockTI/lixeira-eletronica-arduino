# 🗑️ Lixeira Eletrônica com Arduino

Este projeto consiste em uma lixeira eletrônica que utiliza um **sensor de proximidade** para detectar objetos à frente e acionar um **motor servo**, abrindo automaticamente a tampa.

## ⚙️ Funcionamento

1. O sensor de proximidade detecta um objeto à frente.
2. O Arduino aciona o motor servo.
3. A tampa da lixeira permanece aberta enquanto houver algo na frente do sensor.
4. Quando o objeto sai:
   - a lixeira permanece aberta por **10 segundos**
   - após esse tempo, a tampa é fechada automaticamente.

## 🧰 Componentes Utilizados

- Arduino Uno
- Sensor de proximidade (ex: Ultrassônico HC-SR04)
- Motor Servo
- Jumpers
- Protoboard
- Fonte de alimentação

## 📐 Esquema do Projeto

![Esquema](imagens/esquema.jpg)

## 💻 Código

O código-fonte do Arduino está disponível na pasta:

#include <Servo.h>

Servo meuServo;

// Pinos do sensor HC-SR04
const int trigPin = 9; 
const int echoPin = 10; 
// Pino do servo
const int servoPin = 6; 

// Configurações da lixeira
const int distanciaLimite = 20;   // Detecta mão a < 20 cm
const int anguloAberto = 90;      // Ângulo para abrir a tampa
const int anguloFechado = 0;      // Ângulo fechado
const int tempoAberto = 10000;     // Mantém aberto por 10 segundos após sair

bool tampaAberta = false;
unsigned long tempoSaida = 0;

void setup() {
  Serial.begin(9600);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  meuServo.attach(servoPin);
  meuServo.write(anguloFechado);
}

void loop() {
  long duracao, distancia;

  // Envia pulso TRIG
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Lê tempo do retorno no ECHO
  duracao = pulseIn(echoPin, HIGH);

  // Converte para distância
  distancia = duracao * 0.034 / 2;

  Serial.print("Distancia: ");
  Serial.print(distancia);
  Serial.println(" cm");

  if (distancia > 0 && distancia < distanciaLimite) {

    // Detectou objeto
    if (!tampaAberta) {
      abrirTampa();
      tampaAberta = true;
    }

    // Reseta o tempo enquanto tiver algo perto
    tempoSaida = millis();

  } else if (tampaAberta) {

    // Espera 2 segundos após o objeto sair
    if (millis() - tempoSaida >= tempoAberto) {
      fecharTampa();
      tampaAberta = false;
    }
  }

  delay(150);
}

// Função para abrir suavemente
void abrirTampa() {
  for (int pos = anguloFechado; pos <= anguloAberto; pos++) {
    meuServo.write(pos);
    delay(10);
  }
}

// Função para fechar suavemente
void fecharTampa() {
  for (int pos = anguloAberto; pos >= anguloFechado; pos--) {
    meuServo.write(pos);
    delay(50);
  }
}


