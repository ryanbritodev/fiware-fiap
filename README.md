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

### Objetivos:
- Monitorar remotamente a temperatura, umidade e luminosidade de um ambiente.
- Controlar o LED remotamente através de uma interface MQTT.

### Referências:
- **Fiware Descomplicado:** https://github.com/fabiocabrini/fiware
- **Como criar uma VM com o Ubuntu 20.04:** https://youtu.be/X7xwvKdaWqk?si=sMg1QUG8rcGMZNJ3
- **Como rodar o Fiware:** https://youtu.be/q6b1f2CAmno?si=h6AGmgleTCZd_HY-

### Revisões:
- Versão 1.0 - 20/10/2024 - Ryan - Implementação inicial do sistema.
