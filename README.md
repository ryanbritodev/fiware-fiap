# Monitoramento de Temperatura, Umidade, Luminosidade e Controle de LED via MQTT e Fiware
### Participantes:
- Prof. Paulo Marcotti PF2150
- Arthur Cotrick Pagani RM554510
- Diogo Leles Franciulli RM558487
- Felipe Sousa de Oliveira RM559085
- Ryan Brito Pereira Ramos RM554497

### Visão geral 
Este projeto utiliza um ESP32 para monitorar e publicar dados de temperatura, umidade e luminosidade em um broker MQTT 
(via Ubuntu, em uma máquina virtual em uma instância EC2 na AWS). Além disso, o sistema permite controlar o estado de 
um LED remotamente através de um tópico MQTT. Os componentes utilizados são o sensor de temperatura e umidade DHT22, um 
sensor de luminosidade LDR e um LED. A comunicação é realizada via protocolo MQTT, através do Fiware.

### Componentes do Sistema:
1. **ESP32** - Microcontrolador que gerencia a leitura dos sensores e a comunicação MQTT.
2. **DHT22** - Sensor de temperatura e umidade que envia os valores lidos para tópicos MQTT.
3. **LDR (Sensor de Luminosidade)** - Mede a intensidade de luz e envia o valor para um tópico MQTT.
4. **LED** - Controlado remotamente via MQTT. Pode ser ligado ou desligado ao receber comandos.

### Tópicos MQTT Utilizados:
1. **/iot/temperature** - Publica a temperatura lida pelo sensor DHT22.
2. **/iot/humidity** - Publica a umidade relativa lida pelo sensor DHT22.
3. **/iot/luminosity** - Publica o valor de luminosidade lido pelo sensor LDR.
4. **/iot/led** - Tópico utilizado para controlar o LED. 
   - Publicar "on" para ligar o LED.
   - Publicar "off" para desligar o LED.

### Funcionamento:
1. O ESP32 conecta-se a uma rede Wi-Fi e ao broker MQTT, uma máquina virtual com Ubuntu, rodando o Fiware.
2. Ele realiza leituras a cada segundo da temperatura, umidade e luminosidade, publicando os valores nos respectivos tópicos MQTT.
3. Através do tópico /iot/led, é possível enviar comandos para ligar ou desligar o LED remotamente.
4. O sistema pode ser monitorado e controlado através de qualquer aplicativo cliente MQTT, como o MyMQTT.

# Circuito:
<img src="https://github.com/ryanbritodev/fiware-fiap/blob/main/assets/CIRCUITO.png?raw=true"/>

### Link para o Projeto no Wokwi: https://wokwi.com/projects/412207800870787073

# Código do Projeto:
```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

#define DHTPIN 21      // Pino do sensor DHT
#define DHTTYPE DHT22  // Tipo de sensor DHT **OBS: TROCAR PELO DHT11 NO DIA DO CHECKPOINT!!!**
#define LED_PIN 2      // Pino do LED
#define LDR_PIN 34     // Pino do LDR (sensor de luminosidade)

// Informações da rede Wi-Fi
const char* SSID = "";    // Nome da rede Wi-Fi
const char* PASSWORD = "";           // Senha da rede Wi-Fi

// Informações do Broker MQTT
const char* BROKER_MQTT = ""; // IP do Broker MQTT **OBS: PODE MUDAR AO INICIAR A INSTÂNCIA NOVAMENTE!!!**
const int BROKER_PORT = 1883;             // Porta do Broker MQTT
const char* TOPICO_SUBSCRIBE_LED = "/iot/led"; // Tópico para controle do LED
const char* TOPICO_PUBLISH_TEMP = "/iot/temperature"; // Tópico para enviar temperatura
const char* TOPICO_PUBLISH_HUMI = "/iot/humidity";    // Tópico para enviar umidade
const char* TOPICO_PUBLISH_LUX = "/iot/luminosity";   // Tópico para enviar luminosidade
const char* ID_MQTT = "fiware_001"; // ID MQTT

WiFiClient espClient;
PubSubClient MQTT(espClient);
DHT dht(DHTPIN, DHTTYPE);

// Função de verificação da conexão Wi-Fi
void initWiFi() {
  Serial.print("Conectando ao WiFi");
  WiFi.begin(SSID, PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println(" Conectado!");
}

// Função de verificação da conexão ao Broker MQTT
void initMQTT() {
  MQTT.setServer(BROKER_MQTT, BROKER_PORT);
  MQTT.setCallback(mqtt_callback);
}

// Função chamada quando uma mensagem é recebida no MyMQTT
void mqtt_callback(char* topic, byte* payload, unsigned int length) {
  String msg;
  for (int i = 0; i < length; i++) {
    msg += (char)payload[i];
  }
  
  Serial.print("Mensagem recebida no tópico ");
  Serial.print(topic);
  Serial.print(": ");
  Serial.println(msg);

  // Controle do LED via MyMQTT
  if (msg == "on") {
    digitalWrite(LED_PIN, HIGH);
  } else if (msg == "off") {
    digitalWrite(LED_PIN, LOW);
  }
}

// Função para verificar a conexão ao Wi-Fi e MQTT
void VerificaConexoesWiFIEMQTT() {
  if (WiFi.status() != WL_CONNECTED) {
    initWiFi();
  }

  if (!MQTT.connected()) {
    while (!MQTT.connected()) {
      Serial.print("Conectando ao broker MQTT...");
      if (MQTT.connect(ID_MQTT)) {
        Serial.println(" Conectado ao broker MQTT!");
        MQTT.subscribe(TOPICO_SUBSCRIBE_LED);
      } else {
        Serial.print("Falha ao conectar. Erro: ");
        Serial.println(MQTT.state());
        delay(2000);
      }
    }
  }
}

// Publicar dados dos sensores no MyMQTT
void publishSensorData() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
  int ldrValue = analogRead(LDR_PIN);
  int luminosity = map(ldrValue, 0, 4095, 0, 100);  // Mapeia o valor do LDR para 0-100%

  // Publicar temperatura, umidade e luminosidade
  MQTT.publish(TOPICO_PUBLISH_TEMP, String(temperature).c_str());
  MQTT.publish(TOPICO_PUBLISH_HUMI, String(humidity).c_str());
  MQTT.publish(TOPICO_PUBLISH_LUX, String(luminosity).c_str());

  // Log no Monitor Serial
  Serial.print("Temperatura: ");
  Serial.println(temperature);
  Serial.print("Umidade: ");
  Serial.println(humidity);
  Serial.print("Luminosidade: ");
  Serial.println(luminosity);
}

void setup() {
  Serial.begin(115200); // **OBS: CONFIGURAR VELOCIDADE DE BAUDRATE PARA 115200!!!**
  pinMode(LED_PIN, OUTPUT);
  pinMode(LDR_PIN, INPUT);
  dht.begin();

  initWiFi();
  initMQTT();
}

void loop() {
  VerificaConexoesWiFIEMQTT();
  publishSensorData();
  MQTT.loop();
  delay(1000);  // Aguarda 1 segundos entre cada envio de dados
}
```

# Dashboard com o app MyMQTT:
<div display="flex">
<img width="280px" src="https://github.com/ryanbritodev/fiware-fiap/blob/main/assets/DASH%20MYMQTT.jpg?raw=true"/>
<img width="280px" src="https://github.com/ryanbritodev/fiware-fiap/blob/main/assets/T%C3%93PICOS%20PUBLISH.jpg?raw=true"/>
<img width="280px" src="https://github.com/ryanbritodev/fiware-fiap/blob/main/assets/T%C3%93PICOS%20SUBSCRIBE.jpg?raw=true"/>
</div>

### Objetivos:
- Monitorar remotamente a temperatura, umidade e luminosidade de um ambiente.
- Controlar o LED remotamente através de uma interface MQTT.

### Referências:
- **Fiware Descomplicado:** https://github.com/fabiocabrini/fiware
- **Como criar uma VM com o Ubuntu 20.04:** https://youtu.be/X7xwvKdaWqk?si=sMg1QUG8rcGMZNJ3
- **Como rodar o Fiware:** https://youtu.be/q6b1f2CAmno?si=h6AGmgleTCZd_HY-

### Revisões:
- Versão 1.0 - 20/10/2024 - Ryan - Implementação inicial do sistema.
