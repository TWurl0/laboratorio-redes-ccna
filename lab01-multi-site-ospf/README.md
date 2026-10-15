## LAB01-INFRA-CONPORATIVO ##


# Objetivo

O objetivo deste laboratório é projetar e configurar uma infraestrutura de rede funcional que garanta:
- Redundância/Spanning Tree (STP) básica para evitar loops em links redundantes.
- Segmentação de tráfego local via VLANs (Vendas, TI e Gerência).
- Comunicação inter-VLAN utilizando a técnica *Router-on-a-Stick*.
- Roteamento dinâmico entre localidades via **OSPF (Single Area 0)**.
- Distribuição automática de endereços IP via **DHCP Relay**.
- Controle de acesso básico e segurança de borda utilizando **ACLs**.
- Tradução de endereços (**NAT/PAT**) para acesso à Internet simulada.
- Acesso Remoto Seguro (SSH) e Port-Security nos Switches (para demonstrar boas práticas de hardening).


## Topologia

A rede é estruturada em dois sites principais (Matriz e Filial) interconectados por um link serial/Ethernet ponto-a-ponto, com um ponto de saída centralizado para a Internet simulada (ISP).

- **Matriz (HQ):** Roteador R1-HQ conectado ao Switch principal SW-HQ via link Trunk (802.1Q). Atende às VLANs de Vendas e TI.
- **Filial (Branch):** Roteador R2-Branch conectado ao SW-Branch.
- **Link WAN:** Conexão dedicada ponto-a-ponto entre R1-HQ e R2-Branch na sub-rede 10.0.0.0/30 rodando OSPF.
- **Internet (ISP):** Roteador R-ISP conectado ao R1-HQ, atuando como destino final default route (0.0.0.0/0).

<img width="1321" height="743" alt="image" src="https://github.com/user-attachments/assets/3c7b5401-1673-4a44-b0aa-e0791aea1117" />


## Endereçamento IP

A rede utiliza o bloco privado `192.168.0.0/16` para as LANs e `10.0.0.0/30` para a interconexão WAN.

| Dispositivo | Interface | Endereço IP | Máscara | Função / Sub-rede |
| :--- | :--- | :--- | :--- | :--- |
| **R1-HQ** | Gi0/0.10 | 192.168.10.1 | 255.255.255.0 | Gateway VLAN 10 (Vendas) |
| **R1-HQ** | Gi0/0.20 | 192.168.20.1 | 255.255.255.0 | Gateway VLAN 20 (TI) |
| **R1-HQ** | Gi0/1 | 10.0.0.1 | 255.255.255.252 | Link WAN (HQ -> Branch) |
| **R1-HQ** | Gi0/2 | 200.100.10.2 | 255.255.255.252 | Link ISP (Público / NAT) |
| **R2-Branch**| Gi0/0 | 192.168.30.1 | 255.255.255.0 | Gateway VLAN 30 (Filial) |
| **R2-Branch**| Gi0/1 | 10.0.0.2 | 255.255.255.252 | Link WAN (Branch -> HQ) |
| **Servidor DHCP**| Fa10 | 192.168.20.10 | 255.255.255.0 | Servidor central de IPs (VLAN 20) |


## Tecnologias Utilizadas

- **Simulação & Modelagem:** Cisco Packet Tracer 8.x
- **Roteamento:** OSPF (Area 0), Router-on-a-Stick (Inter-VLAN Routing), Static Default Route.
- **Switching:** 802.1Q Trunking, VLANs, Spanning Tree Protocol (STP), Port-Security.
- **Serviços de Infraestrutura:** DHCP Relay (`ip helper-address`), NAT/PAT (Overload), SSH (v2).
- **Segurança:** ACLs Estendidas e Padrão para filtragem de tráfego.


## Implementação

### 1. Configuração de Trunking e Inter-VLAN (R1-HQ e SW-HQ)
Para viabilizar a comunicação entre VLANs, a interface física foi subdividida em sub-interfaces lógicas atreladas ao encapsulamento 802.1Q:

```cisco
! Configuração da Sub-interface VLAN 10 na Matriz
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.20.10
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

## Testes e validação

- [x] Conectividade Local e Remota: `ping` bem-sucedido entre estações da VLAN 10 (Vendas) e VLAN 30 (Filial).

- [x] Concessão de IP Dinâmico: Verificação do recebimento de IP via DHCP nos clientes da VLAN 10 a partir do servidor localizado na VLAN 20.

- [x] Tabela de Roteamento: Checagem da tabela via `show ip route ospf` confirmando o aprendizado das rotas remotas.

- [x] NAT Operacional: Validação da tradução de endereços via `show ip nat translations` durante acessos à Internet simulada.


## Problemas e soluções

**Sintoma:** Dispositivos da VLAN 10 não recebiam IP dinâmico do servidor DHCP na VLAN 20.

**Causa:** O tráfego de broadcast do DHCP foi descartado pelo Roteador na fronteira da VLAN.

**Solução:** Configurado o comando ip helper-address 192.168.20.10 na sub-interface Gi0/0.10, convertendo a requisição DHCP de Broadcast para Unicast em direção ao servidor.


## Aprendizados

Consolidação do funcionamento do DHCP Relay e conversão de pacotes broadcast para unicast através de roteadores.

Ajuste e validação de NAT/PAT de borda combinados com a rota default para saída à Internet.

Importância do Router-on-a-Stick para isolar e controlar a comunicação de segundo nível (Camada 2) antes do encaminhamento de Camada 3.

## Estado do projeto
Concluído!


