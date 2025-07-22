## **Resumo Desafio Eletrônica - Aquisição de Dados**

#### **1. Objetivo do Projeto**
Desenvolvimento de um sistema de aquisição de dados (*datalogger*) para a realização do ensaio de *Coast Down*, com o objetivo de determinar empiricamente as forças resistivas (arrasto aerodinâmico e atrito de rolamento) que atuam sobre um veículo. O projeto deve atender às diretrizes da SAE Brasil e estar em conformidade com a norma ABNT NBR 10312 (2019).

#### **2. Fundamentação Teórica**
O ensaio de *Coast Down* consiste em registrar a velocidade de um veículo durante a desaceleração livre (com a transmissão desacoplada). A desaceleração é governada pela equação de *Road Load*, uma função quadrática da velocidade que soma as resistências ao movimento:
$RL(v) = A + B \cdot v + C \cdot v^2$
Onde os coeficientes A, B e C representam diferentes componentes da força resistiva. Estes coeficientes são determinados experimentalmente através de uma regressão polinomial aplicada aos dados de velocidade coletados.

#### **3. Requisitos de Sistema**
O sistema foi projetado para atender às seguintes especificações mínimas:
* **Frequência de Aquisição:** Superior a 230 Hz para um sinal de onda quadrada proveniente de uma roda fônica de 36 dentes a 40 km/h.
* **Precisão da Velocidade:** Resolução inferior a 0,2 km/h e exatidão de 0,4 km/h, conforme a norma NBR 10312.
* **Armazenamento e Comunicação:** Capacidade de armazenamento local de dados e comunicação sem fio via protocolos ESP-NOW e WiFi.
* **Custo e Peso:** Custo total inferior a R$ 150,00 e peso do invólucro inferior a 0,30 kg.
* **Interface Elétrica:** Compatibilidade com a alimentação de 12V da bateria do veículo e o sinal de 3,3V do sensor.

#### **4. Arquitetura de Hardware**
* **Processador:** Microcontrolador ESP32-WROOM-32, selecionado por sua arquitetura dual-core, que permite o processamento de dados e a comunicação de forma simultânea.
* **Alimentação:** Um conversor *buck* reduz a tensão da bateria de 12V para 3,3V, com proteção de um fusível de 1A.
* **Interface de Sinal:** Um optoacoplador (PC817XNNIPOF) é utilizado para o isolamento elétrico do sinal do sensor (tipo MOSFET). Um resistor de *pull-up* de 560Ω na linha de 12V garante o chaveamento do sinal, que é convertido pelo optoacoplador para o nível lógico de 3,3V do microcontrolador.
* **Armazenamento e Comunicação USB:** O hardware inclui um slot para cartão micro SD, operando via protocolo SPI para redundância de dados, e um chip CP2102 para fornecer uma interface USB para comunicação serial.

#### **5. Arquitetura de Firmware**
* **Estrutura:** Desenvolvido em C++ na Arduino IDE, com uma arquitetura orientada a objetos para modularidade e manutenibilidade.
* **Sistema Operacional:** Utiliza o FreeRTOS para gerenciar tarefas em paralelo nos dois núcleos do ESP32.
* **Núcleo 1:** Dedicado à aquisição de dados. Uma rotina de interrupção (ISR) por borda de subida mede o tempo (Δt) em microssegundos entre os pulsos da roda fônica. Uma tarefa separada calcula a velocidade, aplica um filtro de média móvel e salva os dados em um arquivo CSV no cartão SD.
* **Núcleo 0:** Dedicado à comunicação. Verifica um buffer de dados e, se não estiver vazio, envia as informações para um servidor externo via WiFi (usando protocolo MQTT) ou ESP-NOW.
* **Cálculo da Velocidade:** A velocidade é calculada pela fórmula: $v = (D_{ef} \cdot \pi) / (n \cdot \Delta t)$, onde $D_{ef}$ é o diâmetro efetivo do pneu e n é o número de dentes da roda fônica.

#### **6. Análise de Precisão e Resultados**
A resolução da medição de velocidade foi calculada para a condição mais crítica (40 km/h). Nessa velocidade, o intervalo de tempo entre pulsos é de 4359 µs. Uma variação de 1 µs nesse intervalo resulta em uma resolução de velocidade de aproximadamente 0,007 km/h, um valor significativamente mais preciso que o requisito de 0,2 km/h da norma.
