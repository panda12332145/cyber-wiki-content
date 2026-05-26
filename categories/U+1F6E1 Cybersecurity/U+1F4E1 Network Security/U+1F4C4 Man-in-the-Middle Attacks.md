---
title: Man-in-the-Middle Attacks
difficulty: advanced
starred: true
tags:
  - mitm
  - arp
  - ssl
  - networking
  - cybersecurity
  - packet-sniffing
  - arp-spoofing
  - ssl-stripping
  - network-attacks
  - linux
---

# Man-in-the-Middle Attacks

> Interceptando, monitorando e potencialmente modificando comunicações entre dois dispositivos em uma rede.

<p align="center">
    <img
        src="https://www.imperva.com/learn/wp-content/uploads/sites/13/2019/01/man-in-the-middle-mitm-attack.png"
        width="850">
</p>

---

# O que é um MITM?

MITM (Man-in-the-Middle) é um tipo de ataque onde um invasor posiciona-se entre duas entidades que acreditam estar se comunicando diretamente.

O atacante pode:

- Interceptar tráfego
- Espionar comunicações
- Modificar pacotes
- Redirecionar conexões
- Roubar credenciais
- Injetar payloads maliciosos
- Manipular respostas HTTP/HTTPS

---

# Conceito Visual

```text
[ Vítima ]  <--->  [ Atacante ]  <--->  [ Gateway/Internet ]
```

Sem MITM:

```text
[ Vítima ]  <--->  [ Roteador ]
```

Com MITM:

```text
[ Vítima ]  <--->  [ Atacante ]  <--->  [ Roteador ]
```

O invasor passa a enxergar e controlar o fluxo de dados.

---

# Como o MITM Funciona?

O ataque depende da capacidade do atacante de convencer os dispositivos da rede de que ele é um intermediário legítimo.

Isso normalmente é feito através de:

- ARP Spoofing
- DNS Spoofing
- Rogue Access Points
- Evil Twin
- DHCP Spoofing
- SSL Stripping
- Proxy Injection
- Session Hijacking

---

# Camadas Afetadas

| Técnica            | Camada OSI |
| ------------------ | ---------- |
| ARP Spoofing       | Layer 2    |
| DNS Spoofing       | Layer 7    |
| SSL Stripping      | Layer 7    |
| Rogue AP           | Layer 2    |
| Session Hijacking  | Layer 5-7  |

---

# ARP Spoofing

ARP Spoofing (ARP Poisoning) é uma das técnicas mais clássicas de MITM em redes locais.

O atacante envia respostas ARP falsas para associar seu endereço MAC ao IP do gateway.

---

# O que é ARP?

ARP (Address Resolution Protocol) é utilizado para descobrir o endereço MAC associado a um IP dentro de uma rede local.

## Exemplo

```text
192.168.1.1 -> AA:BB:CC:DD:EE:FF
```

A vítima mantém essas informações em uma tabela ARP.

---

# Funcionamento do ARP Poisoning

O atacante envia respostas ARP falsas:

```text
"192.168.1.1 está no MAC do atacante"
```

e simultaneamente:

```text
"192.168.1.100 está no MAC do atacante"
```

Resultado:

- Vítima envia tráfego para atacante
- Gateway envia tráfego para atacante
- Atacante fica no meio da comunicação

---

# Fluxo do Ataque

```text
+-------------+
|   Vítima    |
+-------------+
       |
       v
+-------------+
|  Atacante   |
+-------------+
       |
       v
+-------------+
|   Gateway   |
+-------------+
```

---

# Descobrindo Hosts na Rede

## Scan ARP

```bash
arp-scan --localnet
```

## Nmap

```bash
nmap -sn 192.168.1.0/24
```

---

# Habilitando IP Forwarding

Para que a vítima continue tendo acesso à internet durante o MITM:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

ou:

```bash
sysctl -w net.ipv4.ip_forward=1
```

---

# ARP Spoofing com arpspoof

```bash
arpspoof -i eth0 -t 192.168.1.100 192.168.1.1
```

```bash
arpspoof -i eth0 -t 192.168.1.1 192.168.1.100
```

## Explicação

| Argumento | Função |
| ---------- | ------- |
| `-i eth0` | Interface de rede |
| `-t` | Alvo |
| `192.168.1.1` | Gateway |
| `192.168.1.100` | Vítima |

---

# Capturando Tráfego

Após o MITM:

```bash
tcpdump -i eth0
```

ou:

```bash
wireshark
```

---

# Protocolos Vulneráveis

Sem criptografia adequada, o atacante pode capturar:

- HTTP
- FTP
- Telnet
- POP3
- SMTP
- IMAP
- Cookies
- Tokens de sessão

---

# Exemplo de Captura HTTP

```http
POST /login HTTP/1.1

username=admin&password=123456
```

Sem HTTPS:

- Credenciais trafegam em texto puro
- Cookies podem ser roubados
- Sessões podem ser sequestradas

---

# SSL Stripping

SSL Stripping força conexões HTTPS a voltarem para HTTP.

O usuário acredita estar seguro, mas o tráfego é descriptografado pelo atacante.

---

# Funcionamento do SSL Strip

```text
Vítima  <----HTTP---->  Atacante  <----HTTPS---->  Site
```

A vítima acessa HTTP.

O atacante:

- intercepta a requisição
- remove redirecionamentos HTTPS
- entrega versão HTTP insegura

---

# SSLStrip

Ferramenta clássica usada para downgrade HTTPS.

## Exemplo

```bash
sslstrip -l 8080 &
```

```bash
iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080
```

---

# Explicando o iptables

```bash
iptables -t nat
```

Manipula NAT.

```bash
-A PREROUTING
```

Intercepta pacotes antes do roteamento.

```bash
--destination-port 80
```

Seleciona tráfego HTTP.

```bash
-j REDIRECT
```

Redireciona pacotes.

---

# Problema do SSL Stripping Moderno

Atualmente muitos sites utilizam:

- HSTS
- HTTPS obrigatório
- Certificate Pinning
- TLS moderno

Isso dificulta ataques SSL Strip tradicionais.

---

# HSTS

HTTP Strict Transport Security força navegadores a utilizarem HTTPS.

Cabeçalho:

```http
Strict-Transport-Security: max-age=31536000
```

---

# DNS Spoofing

Outra técnica comum de MITM.

O atacante responde consultas DNS com IPs falsos.

---

# Exemplo

```text
google.com -> 192.168.1.50
```

A vítima é redirecionada para um servidor malicioso.

---

# Evil Twin

Ataque onde o invasor cria um Access Point falso semelhante ao legítimo.

## Exemplo

```text
WiFi Original: Cafe_Free_Wifi
WiFi Falso:    Cafe_Free_Wifi
```

Usuários conectam-se ao AP malicioso sem perceber.

---

# Captive Portals Maliciosos

Muito utilizados em:

- Aeroportos
- Cafés
- Hotéis
- Eventos

Podem ser usados para:

- Roubo de credenciais
- Coleta de e-mails
- Malware delivery

---

# Session Hijacking

Após capturar cookies:

```http
Cookie: PHPSESSID=abc123
```

O atacante pode assumir sessões autenticadas.

---

# Ferramentas Utilizadas

## Reconhecimento

- Nmap
- arp-scan
- netdiscover

## MITM

- Bettercap
- Ettercap
- arpspoof
- mitmproxy

## Captura

- Wireshark
- tcpdump
- tshark

---

# Bettercap

Framework moderno extremamente poderoso para MITM.

## Exemplo

```bash
bettercap -iface eth0
```

---

# Módulos do Bettercap

- net.probe
- arp.spoof
- http.proxy
- https.proxy
- dns.spoof
- packet.proxy

---

# mitmproxy

Proxy interceptador moderno.

## Exemplo

```bash
mitmproxy
```

Permite:

- editar requests
- modificar responses
- interceptar APIs
- analisar tráfego HTTPS

---

# Detecção de MITM

## Sintomas

- Certificados inválidos
- HTTPS desaparecendo
- Latência estranha
- DNS suspeito
- MAC Address duplicado

---

# Detectando ARP Poisoning

## Linux

```bash
arp -a
```

ou:

```bash
ip neigh
```

---

# Exemplo Suspeito

```text
192.168.1.1 at 00:11:22:33:44:55
192.168.1.100 at 00:11:22:33:44:55
```

Mesmo MAC para múltiplos IPs.

---

# Mitigações

## Rede

- Dynamic ARP Inspection
- VLAN Segmentation
- DHCP Snooping
- Port Security

## Usuário

- Verificar HTTPS
- Utilizar VPN
- Não ignorar alertas TLS
- Evitar Wi-Fi público

---

# Uso de VPN

VPN criptografa o tráfego entre cliente e servidor VPN.

Mesmo sob MITM:

```text
Dados ficam cifrados
```

---

# TLS

TLS protege:

- Confidencialidade
- Integridade
- Autenticação

---

# Fluxo TLS

```text
Client Hello
Server Hello
Certificate
Key Exchange
Encrypted Traffic
```

---

# Certificados Digitais

TLS depende de:

- CA (Certificate Authority)
- Assinaturas digitais
- Chaves públicas/privadas

---

# MITM com Certificados Falsos

Se o usuário aceitar certificados inválidos:

- atacante pode descriptografar tráfego
- proxy TLS pode ser estabelecido

---

# Exemplo com OpenSSL

```bash
openssl s_client -connect example.com:443
```

---

# Wireshark

<p align="center">
    <img
        src="https://www.wireshark.org/docs/wsug_html_chunked/images/ws-main.png"
        width="900">
</p>

---

# Captura de Pacotes

Filtro HTTP:

```text
http
```

Filtro DNS:

```text
dns
```

Filtro TCP:

```text
tcp
```

---

# Exemplo de Análise

Campos importantes:

- Source IP
- Destination IP
- TCP Stream
- HTTP Headers
- TLS Handshake
- DNS Queries

---

# Arquitetura Simplificada do MITM

```text
                +----------------+
                |    Internet    |
                +----------------+
                         ^
                         |
                +----------------+
                |    Gateway     |
                +----------------+
                         ^
                         |
                +----------------+
                |    Atacante    |
                +----------------+
                         ^
                         |
                +----------------+
                |    Vítima      |
                +----------------+
```

---

# Aspectos Técnicos Importantes

## Half Duplex vs Full Duplex

MITM precisa manter comunicação funcional em ambas direções.

---

# Forwarding de Pacotes

Sem forwarding:

```text
Internet cai para vítima
```

Com forwarding:

```text
Tráfego continua funcionando
```

---

# Ataques Modernos Relacionados

- BGP Hijacking
- Rogue DHCP
- WPA2 Enterprise Evil Twin
- KRACK
- DNS Cache Poisoning

---

# Conceitos Fundamentais Relacionados

| Conceito | Relação |
| -------- | ------- |
| TCP/IP | Base da comunicação |
| ARP | Resolução IP/MAC |
| DNS | Resolução de nomes |
| TLS | Criptografia |
| Routing | Encaminhamento |
| NAT | Tradução de endereços |

---

# Exemplo de Laboratório

## Ambiente

```text
Kali Linux -> Atacante
Ubuntu VM  -> Vítima
Router VM   -> Gateway
```

---

# Virtualização

Ferramentas úteis:

- VirtualBox
- VMware
- QEMU/KVM
- Proxmox

---

# Interfaces de Rede

## Bridge Mode

VM participa da rede real.

## NAT

VM usa NAT interno.

## Host-Only

Rede isolada para laboratório.

---

# Conceito de Packet Sniffing

Packet Sniffing é o ato de capturar pacotes da rede.

---

# Promiscuous Mode

Permite à interface capturar pacotes destinados a outros hosts.

---

# Exemplo

```bash
ip link set eth0 promisc on
```

---

# tcpdump

Captura simples via terminal.

## Exemplo

```bash
tcpdump -i eth0 -nn
```

---

# Salvando Captura

```bash
tcpdump -w captura.pcap
```

---

# Abrindo PCAP

```bash
wireshark captura.pcap
```

---

# Estrutura de um Pacote Ethernet

```text
+-------------------+
| Ethernet Header   |
+-------------------+
| IP Header         |
+-------------------+
| TCP/UDP Header    |
+-------------------+
| Payload           |
+-------------------+
```

---

# Conceito de Gateway

Gateway normalmente é o roteador padrão da rede.

---

# Descobrindo Gateway

```bash
ip route
```

---

# Exemplo

```text
default via 192.168.1.1 dev eth0
```

---

# Endereços MAC

MAC é um identificador físico da interface de rede.

## Exemplo

```text
08:00:27:1A:2B:3C
```

---

# Visualizando MAC

```bash
ip link
```

---

# Vídeo

Vídeo:
[https://www.youtube.com/watch?v=-rSqbgI7oZM](https://www.youtube.com/watch?v=-rSqbgI7oZM)

---

# PDF

PDFs:

[https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf](https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf)

https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf

---

# Compatibilidade

| Recurso     | Markdown | GitHub | HTML |
| ----------- | -------- | ------ | ---- |
| Imagem      | ✅        | ✅      | ✅    |
| Vídeo Embed | ❌        | ⚠️     | ✅    |
| iframe      | ❌        | ❌      | ✅    |
| PDF Embed   | ❌        | ❌      | ✅    |
| CSS Inline  | ⚠️       | ⚠️     | ✅    |

---

# Considerações Finais

MITM continua sendo uma das categorias de ataques mais importantes da segurança ofensiva e defensiva.

Mesmo com a evolução do HTTPS, TLS moderno e HSTS, ataques de interceptação continuam relevantes em:

- redes corporativas
- Wi-Fi público
- ambientes internos
- phishing avançado
- proxies maliciosos
- comprometimento de roteadores

Compreender profundamente ARP, DNS, TCP/IP e TLS é fundamental para entender tanto a exploração quanto a defesa contra MITM.

---

# Referências

* https://owasp.org/www-community/attacks/Manipulator-in-the-middle_attack
* https://bettercap.org
* https://www.wireshark.org/docs/
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
* https://datatracker.ietf.org/doc/html/rfc826
* https://www.rfc-editor.org/rfc/rfc8446
* https://www.imperva.com/learn/application-security/man-in-the-middle-attack-mitm/
* https://nmap.org/book/
* https://wiki.wireshark.org/CaptureSetup
* https://tools.kali.org/information-gathering/arp-scan

```
