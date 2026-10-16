# 🌐 Laboratório 03: NAT/PAT, Borda de Rede e DMZ (CCNA 200-301)

## 📌 Visão Geral
Este laboratório aborda a configuração da borda corporativa (*Enterprise Edge*), focando no esgotamento do IPv4 através de técnicas avançadas de **NAT (Static NAT, Dynamic NAT e PAT Overload)**, integração com um Provedor de Serviços (ISP) simulado via **Rota Padrão Estática** e isolamento de serviços públicos em uma zona desmilitarizada (**DMZ**).

---

## 🎯 Objetivos Técnicos
- **NAT Estático (1:1):** Publicação de servidor web interno da DMZ para acesso externo a partir da Internet.
- **PAT Overload (NAT Dinâmico de Porta):** Tradução de múltiplos hosts internos das VLANs para o IP público de borda.
- **Roteamento de Saída:** Rota estática default (`0.0.0.0/0`) direcionada ao gateway do ISP.
- **Zona Desmilitarizada (DMZ):** Segmentação de rede para isolar servidores acessíveis publicamente da rede interna corporativa.
- **Segurança de Borda:** ACLs para permitir tráfego de retorno (inspeção de estado básica) e bloquear conexões não solicitadas vinda do ISP.

---

## 📐 Topologia da Rede
*(Adicione a imagem da topologia aqui: `![Topologia do Lab 03](topologia.png)`)*

- **R1-EDGE:** Roteador de borda corporativo executando NAT/PAT, sub-interfaces e ACLs de segurança.
- **R-ISP:** Roteador simulando a infraestrutura do Provedor de Internet.
- **Server-WEB (DMZ):** Servidor HTTP/HTTPS hospedado na zona DMZ (VLAN 50).
- **PC-LAN:** Estação interna solicitando páginas web e serviços na Internet.
- **Client-External:** Host externo simulando um usuário na Internet acessando a DMZ.

---

## 📊 Tabela de Endereçamento e VLANs

| Dispositivo | Interface | Endereço IP | Máscara | Função / Sub-rede |
| :--- | :--- | :--- | :--- | :--- |
| **R1-EDGE** | Gi0/0.10 | 192.168.10.1 | 255.255.255.0 | Gateway LAN (VLAN 10) |
| **R1-EDGE** | Gi0/0.50 | 192.168.50.1 | 255.255.255.0 | Gateway DMZ (VLAN 50) |
| **R1-EDGE** | Gi0/1 | 203.0.113.2 | 255.255.255.252 | Link WAN / Borda ISP (Público) |
| **R-ISP** | Gi0/0 | 203.0.113.1 | 255.255.255.252 | Gateway do Provedor de Internet |
| **Server-WEB** | Fa0/1 | 192.168.50.10 | 255.255.255.0 | Servidor Web na DMZ |
| **Client-External**| Fa0/1 | 8.8.8.8 | 255.255.255.0 | Host de Testes na Internet |

---

## 🛠️ Tecnologias e Protocolos Utilizados

- **Simulação & Modelagem:** Cisco Packet Tracer 8.x
- **Tradução de Endereços:** Static NAT (1:1), PAT Overload (`ip nat inside source list`).
- **Roteamento de Borda:** Static Default Route (`ip route 0.0.0.0 0.0.0.0`).
- **Segurança & Segmentação:** DMZ (VLAN 50), ACLs de Borda (`Standard/Extended`).
- **Serviços:** HTTP/HTTPS (Porta 80/443), DNS Simulado.

---

## ⚙️ Implementação das Configurações de Segurança

### 1. Definição de Interfaces NAT Inside/Outside (R1-EDGE)
Mapeamento do domínio interno (LAN e DMZ) e externo (Link ISP):

```cisco
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/1
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 no shutdown
2. Configuração de NAT Estático para o Servidor Web (DMZ)
Publicação do IP privado do servidor DMZ (192.168.50.10) no IP público secundário (203.0.113.10):

Cisco CLI
ip nat inside source static 192.168.50.10 203.0.113.10
3. Configuração de PAT Overload para a LAN
Mapeamento de todos os hosts privados internos para o IP da interface pública de saída:

Cisco CLI
access-list 1 permit 192.168.10.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload
4. Rota Padrão de Borda e Roteamento ISP
Cisco CLI
! Rota default em direção ao ISP
ip route 0.0.0.0 0.0.0.0 203.0.113.1

! Rota de retorno no R-ISP para o IP publicado do NAT
ip route 203.0.113.10 255.255.255.255 203.0.113.2
```

✅ Testes e Validação
- [x] Acesso Externo à DMZ (Static NAT): O host Client-External acessou com sucesso a página Web digitando o IP público http://203.0.113.10 no navegador.

- [x] Navegação LAN para Internet (PAT): O PC-LAN realizou pings e conexões com o servidor 8.8.8.8 na Internet simulada.

- [x] Inspeção de Tabela NAT: Comando show ip nat translations executado no R1-EDGE confirmando entradas estáticas de DMZ e dinâmicas de PAT com portas atribuídas.

🔧 Problemas e Soluções
Sintoma: O cliente externo na Internet não conseguia abrir a página web pelo IP público 203.0.113.10.

Causa: O roteador do ISP (R-ISP) não sabia para onde encaminhar os pacotes destinados ao IP público secundário de NAT.

Solução: Adicionada uma rota estática no R-ISP apontando o IP público 203.0.113.10/32 com next-hop na interface WAN do R1-EDGE (203.0.113.2).

📚 Aprendizados
Diferença prática entre Static NAT (para publicar serviços em DMZ) e PAT Overload (para economizar endereços IPv4 na navegação de estações).

Entendimento de rotas de retorno em cenários de NAT de borda com Provedores de Serviço.

Utilização do comando show ip nat translations para monitorar sessões ativas e diagnosticar falhas de conectividade.

🚀 Estado do Projeto
_Em andamento_
