# 🛡️ Laboratório 02: 
_Hardening de Switches e Controle de Acesso (CCNA 200-301)_


## 📌 Visão Geral
Este laboratório foca na aplicação de práticas recomendadas de segurança de infraestrutura de rede (*Hardening*) em dispositivos de comutação (Switches Catalyst) e roteamento Cisco. O projeto aborda a mitigação de ameaças em Camada 2, como *MAC Flooding* e acessos físicos não autorizados, a implementação de acesso remoto seguro via **SSH v2** e o controle de tráfego administrativo usando **ACLs Estendidas**.

---

## 🎯 Objetivos Técnicos
- **Segurança de Porta (Port-Security):** Limitação de endereços MAC por porta, aprendizado dinâmico (*Sticky MAC*) e política de violação (*Shutdown*).
- **Gerenciamento Remoto Seguro:** Configuração de **SSH v2** com encriptação RSA e desativação completa do protocolo não seguro (Telnet).
- **Segmentação Administrativa:** Configuração de SVI (Switched Virtual Interface) em uma VLAN de Gerência dedicada.
- **Filtragem de Tráfego Administrativo:** Aplicação de **ACLs Estendidas** para restringir acessos via SSH e HTTP/HTTPS apenas às estações autorizadas da VLAN de TI.
- **Proteção de Portas Inativas:** Desativação de interfaces não utilizadas e alocação em uma VLAN isolada (*Unused/Blackhole VLAN*).

---

## 📐 Topologia da Rede
<img width="866" height="609" alt="image" src="https://github.com/user-attachments/assets/192177f5-06d7-4524-b57a-93df12d9a947" />

- **SW-Access:** Switch de acesso Catalyst 2960 aplicando políticas de Port-Security e SSH.
- **R1-Core:** Roteador de borda atuando como gateway e aplicando controle de acesso (ACLs).
- **PC-Admin (TI):** Estação de trabalho autorizada a gerenciar os ativos de rede.
- **PC-Invasor/Atacante:** Estação simulando conexão física não autorizada ou tráfego indevido.

---

## 📊 Tabela de Endereçamento e VLANs

| Dispositivo | Interface | Endereço IP | Máscara | VLAN / Função |
| :--- | :--- | :--- | :--- | :--- |
| **SW-Access** | SVI Vlan 99 | 192.168.99.2 | 255.255.255.0 | Gerenciamento (Management) |
| **R1-Core** | Gi0/0.99 | 192.168.99.1 | 255.255.255.0 | Gateway VLAN 99 (Gerência) |
| **R1-Core** | Gi0/0.20 | 192.168.20.1 | 255.255.255.0 | Gateway VLAN 20 (TI) |
| **PC-Admin** | Fa0/1 | 192.168.20.50 | 255.255.255.0 | Estação do Administrador (TI) |
| **PC-User** | Fa0/1 | 192.168.10.50 | 255.255.255.0 | Usuário Padrão (Vendas) |

---

## ⚙️ Implementação das Configurações de Segurança

### 1. Configuração de Port-Security no SW-Access
Mitigação contra conexão de dispositivos não autorizados e ataques de saturação da tabela CAM (*MAC Flooding*):

```cisco
interface FastEthernet0/1
 description CONEXAO-ESTACAO-TI
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky

```

### 2. Hardening de Acesso Remoto (SSH v2) e Desativação do Telnet

Garantia de confidencialidade e integridade no gerenciamento dos switches:

```cisco
ip domain-name empresa.local
crypto key generate rsa general-keys modulus 2048
ip ssh version 2

username admin privilege 15 secret SenhaForte123!

line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0

```

### 3. ACL Estendida para Proteção de Acesso Administrativo (R1-Core)

Restrição para permitir tráfego SSH (porta 22) apenas a partir do IP da estação do Administrador (`192.168.20.50`):

```cisco
ip access-list extended ACL-PROTECT-MGMT
 permit tcp host 192.168.20.50 host 192.168.99.2 eq 22
 permit tcp host 192.168.20.50 host 192.168.99.1 eq 22
 deny tcp any host 192.168.99.2 eq 22
 deny tcp any host 192.168.99.1 eq 22
 permit ip any any

interface GigabitEthernet0/0/0.99
 ip access-group ACL-PROTECT-MGMT out

```

### 4. Desativação de Interfaces Inativas

```cisco
interface range FastEthernet0/5 - 24
 shutdown
 description PORTA-INATIVA-SEGURANCA

```

---

## ✅ Testes e Validação

* [x] **Acesso SSH Seguro:** Conexão SSH estabelecida com sucesso via `PC-Admin` (`192.168.20.50`) para a SVI de gerência (`192.168.99.2`).
* [x] **Bloqueio por ACL:** Tentativas de acesso SSH originadas de estações fora da VLAN de TI descartadas pela ACL.
* [x] **Bloqueio por Port-Security:** Ao trocar a placa de rede/dispositivo na porta `Fa0/1`, a interface mudou imediatamente para o estado `err-disabled` (`violation shutdown`).
* [x] **Inspeção de Telnet:** Conexões Telnet (porta 23) rejeitadas automaticamente pelas linhas VTY do switch.

---

## 🛠️ Problemas e Soluções

* **Sintoma:** Porta em estado `err-disabled` após teste de troca de dispositivo.
* **Causa:** A política de Port-Security detectou um endereço MAC diferente do registrado no comando `sticky`.
* **Solução:** Para restabelecer a porta após a investigação do incidente, executou-se a sequência de recuperação manual no switch:
```cisco
interface FastEthernet0/1
 shutdown
 no shutdown

```



---

## 📚 Aprendizados

* Importância da aplicação de controles de acesso em **Camada 2** para impedir intrusões físicas diretas na rede local.
* Utilização prática de **Port-Security com Sticky MAC** para automatizar o cadastro de dispositivos legítimos.
* Reforço de diretrizes de **Hardening Cisco**, eliminando protocolos em texto claro (Telnet) e restringindo superfícies de ataque via ACLs.

---

## 📋 Estado do Projeto
_Concluído_
