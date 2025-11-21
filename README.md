# Sistema-de-Monitoramento-Card-aco-com-MQTT
🫀 Sistema de Monitoramento Cardíaco com MQTT

Autor: Samuel Couto Vasconcelos
Plataforma: ESP32 (Simulado no Wokwi)
Protocolo: MQTT
Nuvem: Ubidots

📌 Descrição do Projeto

Este projeto implementa um Sistema de Monitoramento Cardíaco IoT, utilizando um ESP32 para:

Gerar sinais simulados de batimentos cardíacos (BPM)

Enviar os dados para a nuvem via MQTT

Receber comandos remotos da plataforma Ubidots

Acionar um buzzer (atuador) via MQTT

O objetivo é demonstrar o funcionamento completo de um sistema IoT usando sensores, atuadores, comunicação TCP/IP e o protocolo MQTT.

🧩 Arquitetura do Sistema

O sistema é composto pelos seguintes módulos:

Módulo Sensor (Simulado):

Gera leituras de BPM entre 60 e 120.

Substitui o sensor físico AD8232.

Módulo de Comunicação (ESP32):

Conecta-se ao Wi-Fi

Publica BPM via MQTT

Assina o tópico de controle do buzzer

Módulo Atuação (Buzzer):

Ativado remotamente por comandos MQTT enviados pelo Ubidots.

🛰️ Fluxo de Funcionamento

O ESP32 inicia

Conecta ao Wi-Fi

Conecta ao broker MQTT do Ubidots

Lê (simula) o valor de BPM

Publica o valor no tópico MQTT

Aguarda comandos

Se receber “1”, aciona o buzzer

Repete continuamente

📡 MQTT – Estrutura dos Tópicos
Publicação (Sensor):
/v1.6/devices/monitoramento-cardiaco/bpm

Subscrição (Atuador):
/v1.6/devices/monitoramento-cardiaco/buzzer/lv

Formato JSON enviado:
{
  "bpm": 82
}

🛠️ Hardware Utilizado

Como o projeto é totalmente simulado, utiliza-se:

ESP32 DevKit V1 (simulado)

Buzzer (simulado)

Sensor cardíaco simulado via código

Ambiente Wokwi

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/19bc5290-835d-4af2-9e22-11b42cab9406" />

💻 Software Necessário

Arduino IDE ou plataforma Wokwi

Conta no Ubidots

Token da Ubidots

Navegador de internet

🧪 Medições de Tempo (Simuladas)
#	Sensor (ms)	Atuador (ms)
1	148	122
2	152	117
3	149	125
4	151	119

Média do sensor: 150 ms

Média do atuador: 120 ms


🔌 Código-Fonte Completo


#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

const char* WIFI_SSID = "DCV";
const char* WIFI_PASS = "ANinha22861959vas@";
const char* UBIDOTS_TOKEN = "BBUS-fnyWnMhF4VfwGIWu4cF10jiFdmR620";

const char* UBIDOTS_DEVICE_LABEL = "monitoramento-cardiaco";

const char* UBIDOTS_BROKER = "industrial.api.ubidots.com";
const int UBIDOTS_PORT = 1883;

const int PIN_BUZZER = 15;

WiFiClient net;
PubSubClient client(net);

unsigned long lastPublish = 0;
const unsigned long PUBLISH_INTERVAL = 2000;

int simulateBPM() {
  float t = millis() / 1000.0;
  float bpm = 80.0 + 15.0 * sin(t/5.0) + (rand() % 7 - 3);
  if (bpm < 40) bpm = 40;
  if (bpm > 200) bpm = 200;
  return (int)bpm;
}

String topicPublish() {
  String t = "/v1.6/devices/";
  t += UBIDOTS_DEVICE_LABEL;
  return t;
}


String topicSubscribe() {
  String t = "/v1.6/devices/";
  t += UBIDOTS_DEVICE_LABEL;
  t += "/buzzer/lv";
  return t;
}

void callback(char* topic, byte* payload, unsigned int length) {
  String msg;

  for (unsigned int i = 0; i < length; i++) {
    msg += (char)payload[i];
  }

  Serial.print("Comando recebido: ");
  Serial.println(msg);

  if (msg == "1") {
    digitalWrite(PIN_BUZZER, HIGH);
    delay(300);
    digitalWrite(PIN_BUZZER, LOW);
  }
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Conectando ao Ubidots MQTT...");
    if (client.connect("esp32_client", UBIDOTS_TOKEN, "")) {
      Serial.println("Conectado!");

      client.subscribe(topicSubscribe().c_str());
      Serial.print("Assinado em: ");
      Serial.println(topicSubscribe());
    }
    else {
      Serial.print("Falha. rc=");
      Serial.println(client.state());
      Serial.println("Tentando novamente em 2s");
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(PIN_BUZZER, OUTPUT);
  digitalWrite(PIN_BUZZER, LOW);

  WiFi.begin(WIFI_SSID, WIFI_PASS);
  Serial.print("Conectando WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");

  client.setServer(UBIDOTS_BROKER, UBIDOTS_PORT);
  client.setCallback(callback);
  randomSeed(analogRead(0));
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastPublish >= PUBLISH_INTERVAL) {
    lastPublish = now;

    int bpm = simulateBPM();


    StaticJsonDocument<150> doc;
    doc["bpm"] = bpm;

    char payload[128];
    serializeJson(doc, payload);

    String pubTopic = topicPublish();
    bool ok = client.publish(pubTopic.c_str(), payload);

    Serial.print("Publicou em ");
    Serial.print(pubTopic);
    Serial.print(" : ");
    Serial.println(payload);
    Serial.println(ok ? "OK" : "ERRO");
  }
}


🌐 Wokwi – Simulação do Projeto

Link oficial do protótipo:

👉 https://wokwi.com/projects/448264081690499073
