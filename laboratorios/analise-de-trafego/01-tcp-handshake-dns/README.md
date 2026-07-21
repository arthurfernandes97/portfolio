<h1 align="center">Análise de Tráfego de Rede: DNS e TCP Handshake</h1>

## Objetivo
Esse laboratório tem como objetivo analisar uma captura de tráfego contendo o acesso ao domínio `www.example.com` via HTTP, focando na resolução de nomes via DNS e no estabelecimento da conexão TCP (Three-Way Handshake).

## Metodologia
Para essa análise usei um arquivo de captura de referência disponibilizado pela comunidade do Wireshark, em vez de gerar uma captura própria.

- **Fonte:** https://wiki.wireshark.org/samplecaptures
- **Arquivo usado:** [netlink-conntrack.pcap](./netlink-conntrack.pcap).
  
  `Arquivo disponível para download neste repositório.`

## Filtragem dos pacotes
Para localizar o tráfego de DNS e TCP mais fácil, apliquei o filtro:
```
tcp || dns
```
Isso reduziu de 107 pacotes para 29, deixando só o que interessava pra análise.

<p align="center">
<img src="imagens/01-filtragem.png" width="1000">
</p>

---

## Fluxo de Análise

### 1. Resolução DNS
Antes de conectar no servidor, o cliente precisa saber o IP do domínio. Pra isso, manda consultas DNS.

**Linhas 9 e 12 - consulta o registro A (IPv4)**

<p align="center">
<img src="imagens/02-query-a.png" width="1000">
</p>

**Linhas 14 e 16 - consulta o registro AAAA (IPv6)**

<p align="center">
<img src="imagens/03-query-aaaa.png" width="1000">
</p>

---

### 2. Three-Way Handshake
Com o IP resolvido, começa o handshake com o servidor na porta 80 (HTTP):

- **22 (SYN):** o cliente pede pra abrir a conexão.
- **25 (SYN, ACK):** o servidor confirma o pedido de conexão do cliente.
- **28 (ACK):** o cliente confirma, e a conexão fica estabelecida.

<p align="center">
<img src="imagens/04-sincronizacao.png" width="1000">
</p>

---

### 3. Requisição HTTP
Com a conexão estabelecida, o cliente manda a primeira requisição. Na linha 29 dá pra ver:
```
GET / HTTP/1.1
Host: www.example.com
```
Como a comunicação utiliza HTTP na porta 80 (sem TLS), o conteúdo trafega em texto claro, sem criptografia.

<p align="center">
<img src="imagens/05-http.png" width="1000">
</p>

---

## Tecnologias utilizadas
- **Wireshark:** ferramenta principal para inspeção de pacotes.
- **Protocolos analisados:** DNS, TCP, HTTP.

## Conclusão
Este laboratório deixou claro o passo a passo de uma conexão web. Primeiro o DNS resolve o domínio por meio de consultas aos registros A (IPv4) e AAAA (IPv6), e só depois disso o cliente inicia o handshake com o servidor.

O Three-Way Handshake (SYN, SYN/ACK, ACK) garante que a conexão TCP está confirmada dos dois lados antes de qualquer dado ser trocado. Só depois do ACK final o cliente manda a requisição HTTP de verdade.

Como essa captura é HTTP puro, sem TLS, deu pra acompanhar tudo isso sem nenhuma criptografia no meio, desde a consulta DNS até o conteúdo da requisição, tudo em texto claro.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
