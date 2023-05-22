# Vaso-de-Plantas-Inteligente
O projeto Vaso de Planta Inteligente Automático é um sistema que visa automatizar a irrigação de plantas em vasos utilizando um microcontrolador Arduino e sensores de umidade no solo.

Materiais necessários:
 O projeto do vaso de planta inteligente automático pode ser construído utilizando os seguintes materiais e métodos:
 
Arduino UNO :É uma placa microcontroladora que pode ser programada para controlar diversos dispositivos, incluindo sensores e atuadores.
 
Cabo jumper para de ligação dos equipamentos: são utilizados para conectar os componentes eletrônicos e garantir o fornecimento de energia elétrica.
 
Placa protoboard: é uma placa de circuito impresso utilizada para montar circuitos eletrônicos experimentais.
 
Sensor de umidade e Placa controladora do sensor de umidade: é um dispositivo que mede a umidade do solo e pode ser utilizado para determinar quando a planta precisa ser irrigada. A placa controladora do sensor de umidade: é uma placa eletrônica que permite conectar o sensor de umidade ao Arduino UNO e processar os dados coletados.
 
Bomba submersa: é uma bomba de água que pode ser submersa no vaso de planta e utilizada para irrigação automática.
 
Relé de energia: é um dispositivo que permite controlar a energia elétrica fornecida à bomba submersa, ligando e desligando a bomba de acordo com as necessidades de irrigação.
 
Fonte de energia USB: É um componente que fornece energia elétrica ao sistema do Arduíno via USB.

Placa ESP01 responsável pela conexão Wifi do arduíno: É um componente que fornecerá a conexão do Arduino com outros aparelhos a internet.
Para construir o sistema de irrigação inteligente, o primeiro passo é conectar o sensor de umidade à placa controladora e a placa controladora ao Arduino UNO. Em seguida, é preciso conectar a bomba submersa ao relé de energia e o relé de energia ao Arduino UNO. Por fim, é necessário programar o Arduino UNO para monitorar os dados do sensor de umidade e acionar a bomba submersa quando necessário, utilizando o relé de energia.

É importante lembrar que o projeto pode ser adaptado de acordo com as necessidades e preferências do usuário, utilizando diferentes sensores, atuadores e componentes eletrônicos.






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
