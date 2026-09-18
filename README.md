# Projeto Prático 5 — Rede de Alta Disponibilidade e Segurança LAN

**Apex Financial — Grupo 7**

Este projeto foi desenvolvido para a disciplina de **Redes de Computadores**, utilizando o **Cisco Packet Tracer** para simular a infraestrutura de rede LAN da empresa fictícia **Apex Financial**.

A proposta principal é desenvolver uma rede com maior disponibilidade, segurança e organização, utilizando recursos como **EtherChannel (Port-Channel)**, **Spanning Tree Protocol (STP)**, **Port Security**, **VLANs**, **roteamento inter-VLAN** e **DHCP**.

A topologia foi pensada considerando que a Apex Financial trabalha com transações financeiras em tempo real. Por isso, a rede precisa continuar funcionando mesmo quando ocorre uma falha em algum dos caminhos de comunicação.

---

# Integrantes do Grupo 7

| Nome completo                        |
| ------------------------------------ |
| Gabriel Henrique da Silva Brandão    |
| Guilherme Araujo Silva               |
| João Victor Carvalho de Faria        |
| João Pedro Gonsalves de Aquino       |
| Vinicius Medeiros de Carvalho Santos |
| André Luis                           |
| Arthur Moisés                        |

**Disciplina:** Redes de Computadores
**Professor orientador:** Hudson Neves e Silva

---

# 1. Sobre o cenário da empresa

A **Apex Financial** é uma empresa fictícia do setor financeiro que realiza transações em tempo real. Nesse tipo de ambiente, a disponibilidade da rede é muito importante, pois uma interrupção pode afetar diretamente as operações da empresa.

Além da disponibilidade, também existe a preocupação com a segurança. Como vários computadores estão conectados à rede local, é necessário controlar os dispositivos que podem utilizar as portas dos switches.

Para atender a essas necessidades, a rede foi montada utilizando diferentes recursos de infraestrutura e segurança, buscando evitar pontos únicos de falha e reduzir os riscos de indisponibilidade.

---

# 2. Estrutura da topologia

A topologia foi organizada principalmente em duas camadas: **Núcleo (Core)** e **Acesso**.

### 2.1 Núcleo (Core)

O núcleo é formado pelos switches multicamada:

* `Multilayer Switch2`
* `Multilayer Switch3`

Esses dois equipamentos possuem a função de realizar o **roteamento entre as VLANs**.

Eles também estão conectados por meio de um **Port-Channel utilizando LACP**, permitindo que mais de um link físico seja utilizado como uma conexão lógica. Dessa forma, é possível aumentar a capacidade do enlace e também melhorar a disponibilidade da comunicação entre os switches do núcleo.

### 2.2 Camada de Acesso

A camada de acesso é formada por dois switches de Camada 2:

* `Switch0`
* `Switch1`

Cada switch de acesso possui conexões com os dois switches do núcleo. Também existe uma conexão entre os próprios switches de acesso.

Essa estrutura cria caminhos físicos redundantes. Para evitar que esses caminhos formem loops na rede, foi utilizado o **Spanning Tree Protocol (STP)**.

---

# 3. Dispositivos utilizados

A topologia conta com os seguintes dispositivos:

* `Server2`
* `Server3`
* `Laptop2`
* `Laptop3`
* `Laptop4`
* `Laptop5`
* `PC0` até `PC19`
* `Switch0`
* `Switch1`
* `Multilayer Switch2`
* `Multilayer Switch3`

Os servidores fazem parte da infraestrutura da rede, enquanto os laptops são utilizados como estações de gerência.

Os 20 computadores de operadores estão distribuídos entre os switches `Switch0` e `Switch1`.

---

# 4. Endereçamento IP e VLANs

A rede foi dividida em três VLANs, cada uma com uma função específica. Essa separação ajuda a organizar os dispositivos e também permite controlar melhor o tráfego da rede.

| VLAN                               | Sub-rede       | Gateway padrão | Função / Dispositivos associados                                | Portas de switch alocadas                                                                                   |
| ---------------------------------- | -------------- | -------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **VLAN 100 – Núcleo e Servidores** | 10.100.10.0/24 | 10.100.10.1    | Server2, Server3, Laptop2 e Laptop3                             | Multilayer Switch3: Gi0/1 (Server2), Gi0/2 (Laptop3) · Multilayer Switch2: Gi0/1 (Server3), Gi0/2 (Laptop2) |
| **VLAN 101 – Operação Norte**      | 10.100.20.0/24 | 10.100.20.1    | PC0, PC1, PC2, PC3, PC4, PC13, PC17, PC10, PC11, PC12 e Laptop5 | Switch1: Fa0/1–Fa0/10 (PCs), Fa0/11 (Laptop5)                                                               |
| **VLAN 102 – Operação Sul**        | 10.100.30.0/24 | 10.100.30.1    | PC18, PC16, PC15, PC14, PC5, PC6, PC7, PC19, PC8, PC9 e Laptop4 | Switch0: Fa0/1–Fa0/10 (PCs), Fa0/11 (Laptop4)                                                               |

O backbone entre `Multilayer Switch2` e `Multilayer Switch3` utiliza um **Port-Channel em modo trunk**, permitindo as VLANs **100, 101 e 102**.

Os links entre os switches de acesso e os switches do núcleo também funcionam como trunks. Nesse caso, o **STP** é responsável por controlar os caminhos e impedir a formação de loops.

---

# 5. Configurações principais

A seguir estão os principais recursos utilizados na configuração da rede.

### 5.1 Criação das VLANs

Nos switches `Switch0` e `Switch1`, foram criadas as três VLANs utilizadas no projeto:

```text
Switch(config)# vlan 100
Switch(config-vlan)# name NUCLEO_SERVIDORES
Switch(config-vlan)# exit

Switch(config)# vlan 101
Switch(config-vlan)# name OPERACAO_NORTE
Switch(config-vlan)# exit

Switch(config)# vlan 102
Switch(config-vlan)# name OPERACAO_SUL
Switch(config-vlan)# exit
```

As VLANs permitem separar logicamente os diferentes setores da rede, mesmo utilizando a mesma infraestrutura física.

---

### 5.2 Roteamento entre VLANs

Nos switches multicamada, foi habilitado o roteamento IP e configuradas as interfaces virtuais de cada VLAN:

```text
Switch(config)# ip routing

Switch(config)# interface vlan 100
Switch(config-if)# ip address 10.100.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch(config)# interface vlan 101
Switch(config-if)# ip address 10.100.20.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch(config)# interface vlan 102
Switch(config-if)# ip address 10.100.30.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

Com essa configuração, os dispositivos de diferentes VLANs conseguem se comunicar por meio dos switches multicamada.

---

### 5.3 EtherChannel / Port-Channel com LACP

A conexão entre os dois switches do núcleo foi configurada utilizando **EtherChannel**, com **LACP** para agrupar os links físicos em uma única conexão lógica:

```text
Switch(config)# interface range gigabitEthernet 0/1-2
Switch(config-if-range)# channel-group 1 mode active
Switch(config-if-range)# exit

Switch(config)# interface port-channel 1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 100,101,102
```

Essa configuração permite utilizar os links de forma agregada, proporcionando maior capacidade de comunicação e redundância entre os equipamentos do núcleo.

---

### 5.4 Spanning Tree Protocol (STP)

Para controlar os caminhos redundantes da rede, foi utilizado o **Spanning Tree Protocol**.

O `Multilayer Switch3` foi definido como **root primary**, enquanto o `Multilayer Switch2` foi configurado como **root secondary**:

```text
Multilayer_Switch3(config)# spanning-tree vlan 100,101,102 root primary

Multilayer_Switch2(config)# spanning-tree vlan 100,101,102 root secondary
```

O STP permite que a rede tenha caminhos redundantes sem criar loops. Quando necessário, alguns caminhos podem permanecer bloqueados até que sejam necessários para a comunicação.

---

### 5.5 Port Security

As portas de acesso utilizadas pelos computadores foram configuradas com **Port Security**.

O objetivo é permitir apenas um endereço MAC por porta e registrar automaticamente o endereço autorizado utilizando o recurso de **MAC Sticky**.

```text
Switch(config)# interface range fastEthernet 0/1-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport port-security
Switch(config-if-range)# switchport port-security maximum 1
Switch(config-if-range)# switchport port-security violation shutdown
Switch(config-if-range)# switchport port-security mac-address sticky
```

Caso um dispositivo diferente seja conectado a uma porta que já possui um endereço MAC autorizado, a violação de segurança fará com que a porta seja desativada.

---

### 5.6 DHCP

O DHCP foi configurado para distribuir automaticamente os endereços IP para os dispositivos das VLANs de operação.

```text
Switch(config)# ip dhcp pool VLAN101_OPERACAO_NORTE
Switch(dhcp-config)# network 10.100.20.0 255.255.255.0
Switch(dhcp-config)# default-router 10.100.20.1
Switch(dhcp-config)# exit

Switch(config)# ip dhcp pool VLAN102_OPERACAO_SUL
Switch(dhcp-config)# network 10.100.30.0 255.255.255.0
Switch(dhcp-config)# default-router 10.100.30.1
Switch(dhcp-config)# exit

Switch(config)# ip dhcp excluded-address 10.100.20.1
Switch(config)# ip dhcp excluded-address 10.100.30.1
```

Dessa forma, os computadores recebem automaticamente as informações necessárias para acessar a rede, evitando a necessidade de configurar cada endereço IP manualmente.

---

# 6. Testes de funcionamento

Depois de realizar as configurações, foram definidos alguns testes para verificar o funcionamento dos principais recursos da rede.

### 6.1 Verificação das VLANs

No `Switch0` e no `Switch1`, é possível verificar se as VLANs foram criadas corretamente e se os computadores estão conectados às portas correspondentes.

A expectativa é que:

* a VLAN 100 seja utilizada para o núcleo e os servidores;
* a VLAN 101 seja utilizada pela Operação Norte;
* a VLAN 102 seja utilizada pela Operação Sul.

---

### 6.2 Verificação do Port-Channel

No `Multilayer Switch2` e no `Multilayer Switch3`, deve ser possível verificar se os dois links utilizados no núcleo estão funcionando como um único **Port-Channel**.

O objetivo é confirmar que o LACP conseguiu agrupar corretamente as interfaces configuradas.

---

### 6.3 Verificação do roteamento entre VLANs

Para testar o roteamento, deve ser realizada uma comunicação entre dispositivos que estejam em VLANs diferentes.

Por exemplo, um computador da **VLAN 101** pode tentar se comunicar com outro computador da **VLAN 102**.

Se a comunicação ocorrer normalmente, significa que o roteamento entre as VLANs está funcionando.

---

### 6.4 Verificação do Spanning Tree

A topologia possui vários caminhos físicos entre os switches. Por isso, o STP deve manter o funcionamento da rede sem permitir a criação de loops.

Durante o teste, deve ser possível identificar o caminho ativo e os caminhos que permanecem bloqueados pelo STP.

---

### 6.5 Teste do Port Security

Para testar a segurança das portas:

1. Escolha um computador conectado ao `Switch0` ou `Switch1`.
2. Desconecte o computador autorizado.
3. Conecte outro dispositivo na mesma porta.
4. Verifique o comportamento da interface.

Como a porta foi configurada para permitir apenas um endereço MAC, a conexão de um dispositivo diferente deve gerar uma violação e fazer com que a porta seja desativada.

---

### 6.6 Teste do DHCP

Em qualquer computador da rede:

1. Clique no PC.
2. Acesse **Desktop**.
3. Entre em **IP Configuration**.
4. Selecione **DHCP**.
5. Aguarde a obtenção automática das informações de rede.

O computador deverá receber um endereço IP pertencente à rede da VLAN em que está conectado.

Por exemplo:

* **VLAN 101:** `10.100.20.0/24`
* **VLAN 102:** `10.100.30.0/24`

---

### 6.7 Teste de comunicação entre VLANs

Em um computador da **VLAN 101**, abra o **Command Prompt** e faça um ping para um computador da **VLAN 102**:

```text
ping <IP de um PC da VLAN 102>
```

Se o roteamento estiver funcionando corretamente, o computador deverá receber respostas do dispositivo de destino.

Esse teste confirma que a comunicação entre as diferentes VLANs está funcionando por meio dos switches multicamada.

---

### 6.8 Teste de redundância

Para verificar a capacidade de recuperação da rede:

1. Coloque o Packet Tracer no modo **Realtime**.
2. Em um PC, inicie um ping contínuo para um servidor do núcleo:

```text
ping -t <IP do Server2 ou Server3>
```

3. Enquanto o ping estiver sendo executado, remova fisicamente um dos cabos que conecta um switch de acesso a um dos switches do núcleo.
4. Observe o comportamento da comunicação.

O objetivo é verificar se a rede consegue utilizar o caminho alternativo disponível na topologia e manter a comunicação mesmo após a falha de um dos enlaces.

---

# 7. Observações importantes

* Os nomes e números das interfaces apresentados neste README devem ser conferidos no arquivo `.pkt`, pois podem variar de acordo com a versão do Packet Tracer ou com a montagem da topologia.
* A identificação das portas (`Fa0/1`, `Gi0/1`, etc.) deve ser comparada com a topologia utilizada no projeto.
* Os testes devem ser realizados somente depois que todos os equipamentos estiverem configurados e com as interfaces ativas.
* Para os testes de segurança e redundância, é importante observar o comportamento da rede durante a simulação.
* O arquivo `.pkt` é a principal referência para conferir a montagem física e a configuração utilizada no projeto.

---

# 8. Objetivo final do projeto

De forma geral, o projeto busca demonstrar como diferentes recursos de redes podem trabalhar juntos para atender às necessidades de uma empresa que depende de **disponibilidade, organização e segurança**.

A utilização de **VLANs** organiza a rede, o **roteamento inter-VLAN** permite a comunicação entre diferentes segmentos, o **EtherChannel** proporciona uma conexão agregada entre os equipamentos do núcleo, o **STP** evita loops nos caminhos redundantes, o **Port Security** ajuda a controlar o acesso físico às portas e o **DHCP** facilita a distribuição dos endereços IP.

Com a combinação desses recursos, a topologia da Apex Financial representa uma infraestrutura LAN mais organizada, redundante e preparada para lidar com falhas de conexão e tentativas de acesso não autorizado.
