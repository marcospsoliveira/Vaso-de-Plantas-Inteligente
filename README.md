# Vaso-de-Plantas-Inteligente
O projeto Vaso de Planta Inteligente Automático é um sistema que visa automatizar a irrigação de plantas em vasos utilizando um microcontrolador Arduino e sensores de umidade no solo. 
\\\código sem conexão com o WIFI.
#define rele 3
#define sensor 2
bool irrigar = false;
void setup()
{
  pinMode(rele, OUTPUT);
  pinMode(sensor, INPUT);
  
  digitalWrite( rele, HIGH);
}
void loop() 
{
  irrigar = digitalRead(sensor);

  if(irrigar==false)
  {
    digitalWrite(rele, HIGH);
  }
  else
  {
    digitalWrite(rele, LOW);
    delay(1500);
    digitalWrite(rele, HIGH);
  }
  delay(10000);
}

\\\código para a conexão com MQTT
#include <PubSubClient.h>
#include <WiFi.h>

// Dados da rede WiFi
const char* ssid = "nome_da_rede";
const char* password = "senha_da_rede";

// Dados do servidor MQTT
const char* mqttServer = "endereço_do_servidor_mqtt";
const int mqttPort = 1883;
const char* mqttUser = "usuário_mqtt";
const char* mqttPassword = "senha_mqtt";

#define rele 3
#define sensor 2
bool irrigar = false;

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  pinMode(rele, OUTPUT);
  pinMode(sensor, INPUT);
  
  digitalWrite(rele, HIGH);
  
  // Conecta-se à rede WiFi
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
  }

  // Configura o servidor MQTT
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  // Conecta-se ao servidor MQTT
  while (!client.connected()) {
    if (client.connect("arduinoClient", mqttUser, mqttPassword)) {
      client.subscribe("irrigar");
    }
    else {
      delay(1000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  irrigar = digitalRead(sensor);

  if (irrigar == false) {
    digitalWrite(rele, HIGH);
  }
  else {
    digitalWrite(rele, LOW);
    delay(1500);
    digitalWrite(rele, HIGH);
  }
  delay(10000);
}

void callback(char* topic, byte* payload, unsigned int length) {
  // Lida com mensagens recebidas pelo tópico "irrigar"
  if (strcmp(topic, "irrigar") == 0) {
    if (payload[0] == '1') {
      digitalWrite(rele, LOW);
      delay(1500);
      digitalWrite(rele, HIGH);
    }
  }
}

void reconnect() {
  // Reconecta-se ao servidor MQTT
  while (!client.connected()) {
    if (client.connect("arduinoClient", mqttUser, mqttPassword)) {
      client.subscribe("irrigar");
    }
    else {
      delay(1000);
    }
  }}
