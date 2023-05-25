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



Código Arduino UNO
#define rele 3
#define sensor 1
#define sensor2 2
#define botao 8
bool irrigar = false;
bool botaoPressionado = false;
bool sensor2SinalAnterior = false;

void setup()
{
  pinMode(rele, OUTPUT);
  pinMode(sensor, INPUT);
  pinMode(sensor2, INPUT);
  pinMode(botao, INPUT);
  
  digitalWrite(rele, HIGH);
  
  Serial.begin(115200); // Inicia a comunicação serial com uma taxa de 115200 bps
}

void loop() 
{
  irrigar = digitalRead(sensor);
  botaoPressionado = digitalRead(botao);
  bool sensor2SinalAtual = digitalRead(sensor2);

  if (irrigar == false || sensor2SinalAtual == HIGH)
  {
    digitalWrite(rele, HIGH);
    Serial.println("Irrigar: Desligado");
  }
  else if (botaoPressionado == true)
  {
    digitalWrite(rele, LOW);
    Serial.println("Irrigar: Ligado");
  }
  else
  {
    digitalWrite(rele, HIGH);
    Serial.println("Irrigar: Desligado");
  }
  
  if (sensor2SinalAtual != sensor2SinalAnterior)
  {
    if (sensor2SinalAtual == HIGH)
    {
      digitalWrite(rele, LOW);
      delay(1800);
      digitalWrite(rele, HIGH);
      delay(10000);
      digitalWrite(rele, LOW);
      Serial.println("Sensor 2: Sinal detectado");
    }
    else
    {
      Serial.println("Sensor 2: Sinal perdido");
      digitalWrite(rele, HIGH); // Ativa o relé quando o sinal do sensor 2 for perdido
    }
    delay(1800);
    sensor2SinalAnterior = sensor2SinalAnterior;
  }
  
  delay(800);
}



\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

Código da placa responsável pelo envio dos dados

#include <ESP8266WiFi.h> 
#include <PubSubClient.h>

#define pinBotao1 12  //D6

//WiFi
const char* SSID = "Galaxy A30s Marcos 4";                // SSID / nome da rede WiFi que deseja se conectar
const char* PASSWORD = "1234567890";   // Senha da rede WiFi que deseja se conectar
WiFiClient wifiClient;                        
 
//MQTT Server
const char* BROKER_MQTT = "mqtt.eclipseprojects.io"; //URL do broker MQTT que se deseja utilizar
int BROKER_PORT = 1883;                      // Porta do Broker MQTT

#define ID_MQTT  "MARCOSESPD0A2E0"            //Informe um ID unico e seu. Caso sejam usados IDs repetidos a ultima conexão irá sobrepor a anterior. 
#define TOPIC_PUBLISH "MARCOSESP2D0A2E0Botao1"    //Informe um Tópico único. Caso sejam usados tópicos em duplicidade, o último irá eliminar o anterior.
PubSubClient MQTT(wifiClient);        // Instancia o Cliente MQTT passando o objeto espClient

//Declaração das Funções
void mantemConexoes();  //Garante que as conexoes com WiFi e MQTT Broker se mantenham ativas
void conectaWiFi();     //Faz conexão com WiFi
void conectaMQTT();     //Faz conexão com Broker MQTT
void enviaValores();     //

void setup() {
  pinMode(pinBotao1, INPUT_PULLUP);         

  Serial.begin(115200);

  conectaWiFi();
  MQTT.setServer(BROKER_MQTT, BROKER_PORT);   
}

void loop() {
  mantemConexoes();
  enviaValores();
  MQTT.loop();
}

void mantemConexoes() {
    if (!MQTT.connected()) {
       conectaMQTT(); 
    }
    
    conectaWiFi(); //se não há conexão com o WiFI, a conexão é refeita
}

void conectaWiFi() {

  if (WiFi.status() == WL_CONNECTED) {
     return;
  }
        
  Serial.print("Conectando-se na rede: ");
  Serial.print(SSID);
  Serial.println("  Aguarde!");

  WiFi.begin(SSID, PASSWORD); // Conecta na rede WI-FI  
  while (WiFi.status() != WL_CONNECTED) {
      delay(100);
      Serial.print(".");
  }
  
  Serial.println();
  Serial.print("Conectado com sucesso, na rede: ");
  Serial.print(SSID);  
  Serial.print("  IP obtido: ");
  Serial.println(WiFi.localIP()); 
}

void conectaMQTT() { 
    while (!MQTT.connected()) {
        Serial.print("Conectando ao Broker MQTT: ");
        Serial.println(BROKER_MQTT);
        if (MQTT.connect(ID_MQTT)) {
            Serial.println("Conectado ao Broker com sucesso!");
        } 
        else {
            Serial.println("Noo foi possivel se conectar ao broker.");
            Serial.println("Nova tentatica de conexao em 10s");
            delay(10000);
        }
    }
}

void enviaValores() {
static bool estadoBotao1 = HIGH;
static bool estadoBotao1Ant = HIGH;
static unsigned long debounceBotao1;

  estadoBotao1 = digitalRead(pinBotao1);
  if (  (millis() - debounceBotao1) > 30 ) {  //Elimina efeito Bouncing
     if (!estadoBotao1 && estadoBotao1Ant) {

        //Botao Apertado     
        MQTT.publish(TOPIC_PUBLISH, "1");
        Serial.println("Botao1 APERTADO. Payload enviado.");
        
        debounceBotao1 = millis();
     } else if (estadoBotao1 && !estadoBotao1Ant) {

        //Botao Solto
        MQTT.publish(TOPIC_PUBLISH, "0");
        Serial.println("Botao1 SOLTO. Payload enviado.");
        
        debounceBotao1 = millis();
     }
     
  }
  estadoBotao1Ant = estadoBotao1;
}




\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\


Código da placa responsável pelo recebimento dos dados:

  #include <ESP8266WiFi.h> 
  #include <PubSubClient.h>

  #define pinLED1 12  //D6

  //WiFi
  const char* SSID = "Galaxy A30s Marcos 4";                // SSID / nome da rede WiFi que deseja se conectar
  const char* PASSWORD = "1234567890";   // Senha da rede WiFi que deseja se conectar
  WiFiClient wifiClient;                        
  
  //MQTT Server
  const char* BROKER_MQTT = "mqtt.eclipseprojects.io"; //URL do broker MQTT que se deseja utilizar
  int BROKER_PORT = 1883;                      // Porta do Broker MQTT

  #define ID_MQTT  "MARCOSESP2D0A2E0"            //Informe um ID unico e seu. Caso sejam usados IDs repetidos a ultima conexão irá sobrepor a anterior. 
  #define TOPIC_PUBLISH "MARCOSESP2D0A2E0Botao1"    //Informe um Tópico único. Caso sejam usados tópicos em duplicidade, o último irá eliminar o anterior.
  PubSubClient MQTT(wifiClient);        // Instancia o Cliente MQTT passando o objeto espClient

  //Declaração das Funções
  void mantemConexoes();  //Garante que as conexoes com WiFi e MQTT Broker se mantenham ativas
  void conectaWiFi();     //Faz conexão com WiFi
  void conectaMQTT();     //Faz conexão com Broker MQTT
  void recebePacote(char* topic, byte* payload, unsigned int length);

  void setup() {
    pinMode(pinLED1, OUTPUT);         

    Serial.begin(115200);

    conectaWiFi();
    MQTT.setServer(BROKER_MQTT, BROKER_PORT);   
    MQTT.setCallback(recebePacote);
  }

  void loop() {
    mantemConexoes();
    MQTT.loop();
  }

  void mantemConexoes() {
      if (!MQTT.connected()) {
        conectaMQTT(); 
      }
      
      conectaWiFi(); //se não há conexão com o WiFI, a conexão é refeita
  }

  void conectaWiFi() {

    if (WiFi.status() == WL_CONNECTED) {
      return;
    }
          
    Serial.print("Conectando-se na rede: ");
    Serial.print(SSID);
    Serial.println("  Aguarde!");

    WiFi.begin(SSID, PASSWORD); // Conecta na rede WI-FI  
    while (WiFi.status() != WL_CONNECTED) {
        delay(100);
        Serial.print(".");
    }
    
    Serial.println();
    Serial.print("Conectado com sucesso, na rede: ");
    Serial.print(SSID);  
    Serial.print("  IP obtido: ");
    Serial.println(WiFi.localIP()); 
  }

  void conectaMQTT() { 
      while (!MQTT.connected()) {
          Serial.print("Conectando ao Broker MQTT: ");
          Serial.println(BROKER_MQTT);
          if (MQTT.connect(ID_MQTT)) {
              Serial.println("Conectado ao Broker com sucesso!");
          } 
          else {
              Serial.println("Noo foi possivel se conectar ao broker.");
              Serial.println("Nova tentatica de conexao em 10s");
              delay(10000);
          }
      }
  }

  void recebePacote(char* topic, byte* payload, unsigned int length)
  {String msg;
  //obtem a string do payload recebido
  for(int i = 0; i< length; i++)
  {
      char c = (char)payload[i];
      msg += c;
    Serial.print("recebido mensagem payload do botão");
  }

  if (msg == "0"){
    digitalWrite(pinLED1, LOW);
  Serial.print("recebido aperto o botão");
  }

  if (msg == "1"){
    digitalWrite(pinLED1, HIGH);
  Serial.print("recebido levantamento do botão");
  }

  }
