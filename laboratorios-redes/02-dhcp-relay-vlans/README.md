<h1 align="center">DHCP Relay com Múltiplas VLANs</h1>

## Objetivo
Esse laboratório tem como objetivo montar uma rede com múltiplas VLANs, cada uma em uma sub-rede diferente, e configurar um servidor DHCP centralizado que atenda todas elas através de DHCP Relay (`ip helper-address`), já que requisições DHCP não atravessam VLANs diferentes por padrão.

## Tecnologias utilizadas
- Cisco Packet Tracer
- VLANs, Trunk, Router-on-a-Stick
- DHCP Relay (`ip helper-address`)

## Topologia
A rede tem 4 VLANs: Vendas (172.16.10.0/24), Administração (172.16.20.0/24), TI (172.16.30.0/24) e Servidor (172.16.40.0/24). As três primeiras têm PCs e laptops, a última só o servidor DHCP.

O roteador fica conectado a dois switches, cada um levando duas VLANs. A ideia é configurar esses links como trunk e criar uma subinterface por VLAN no roteador (router-on-a-stick). Nesse print a topologia ainda está sem nenhuma configuração, os links aparecem sem conectividade porque nada tinha sido configurado ainda.

<p align="center">
<img src="images/01-topologia.png" width="850">
</p>

---

## Configuração dos switches
Em cada switch, criei as VLANs, coloquei as portas dos hosts em modo access na VLAN correspondente, e configurei a porta que liga ao roteador como trunk, permitindo só as VLANs daquele lado.

**Switch 1 (Vendas e Administração):**

<p align="center">
<img src="images/02-config-switch1.png" width="950">
</p>

---

**Switch 2 (TI e Servidor):**

<p align="center">
<img src="images/03-config-switch2.png" width="950">
</p>

---

## Configuração do roteador
Criei uma subinterface por VLAN, cada uma com `encapsulation dot1Q` e o IP de gateway daquela rede. Nas subinterfaces das VLANs de cliente (10, 20 e 30), adicionei o `ip helper-address` apontando pro IP do servidor DHCP, pra encaminhar o broadcast das requisições até ele.

<p align="center">
<img src="images/04-config-router.png" width="600">
</p>

Nesse processo tive dois erros de digitação/sintaxe que valem registrar:

**1.** Na subinterface da VLAN 40, tentei configurar o `ip address` antes do `encapsulation dot1Q`, e o IOS recusou:
```
% Configuring IP routing on a LAN subinterface is only allowed if that
subinterface is already configured as part of an IEEE 802.1Q, IEEE 802.1Q,
or ISL vLAN.
```
Faz sentido: sem o encapsulamento definido, a subinterface não pertence a nenhuma VLAN ainda, então o IOS não sabe em que rede lógica aquele IP deveria existir. Corrigi rodando o `encapsulation dot1Q 40` primeiro e repetindo o `ip address` depois.

**2.** Mais tarde, tentei rodar `show ip interface g0/0/1.30` direto dentro do modo de configuração, e recebi:
```
% Invalid input detected at '^' marker.
```
Comando de modo EXEC (como `show`) não roda direto dentro do modo `config` — precisa do prefixo `do`. Corrigi com `do show ip interface g0/0/1.30` e funcionou.

## Configuração do servidor DHCP
Configurei o IP estático do servidor, e os três pools (um por VLAN de cliente), cada um com o gateway, a faixa de IPs e a máscara certos.

<p align="center">
<img src="images/05-ip-servidor.png" width="500">
</p>

<p align="center">
<img src="images/06-config-servidor-dhcp.png" width="1000">
</p>

---

## Testes iniciais
VLAN 10 e VLAN 20 pegaram IP via DHCP sem problema, confirmando que o relay estava funcionando nessas duas.

<p align="center">
<img src="images/07-teste-dhcp-vlan10.png" width="1000">
</p>

<p align="center">
<img src="images/08-teste-dhcp-vlan20.png" width="1000">
</p>

---

## Troubleshooting: VLAN 30 sem IP
Na VLAN 30 (TI), os hosts não conseguiram pegar IP:
```
DHCP request failed.
```

<p align="center">
<img src="images/09-erro-dhcp-vlan30.png" width="1000">
</p>

Fui comparar a configuração da subinterface `g0/0/1.30` com as outras que estavam funcionando, usando:
```
do show ip interface g0/0/1.30
```
E o resultado mostrou:
```
Helper address is not set
```
Diferente das subinterfaces 10 e 20, a 30 estava sem o `ip helper-address`. Sem ele, o broadcast da requisição DHCP nunca chegava até o servidor, que está em outra rede. Configurei o helper que faltava:
```
ip helper-address 172.16.40.2
```

<p align="center">
<img src="images/10-solucionando-erro.png" width="600">
</p>

---

## Teste da VLAN 30 corrigida
Depois do ajuste, os hosts da VLAN 30 conseguiram pegar IP normalmente.

<p align="center">
<img src="images/11-teste-dhcp-vlan30.png" width="1000">
</p>

---

## Teste de conectividade final
Com as 4 redes funcionando, testei ping do PC1 (VLAN 10) até um host de cada VLAN, incluindo o servidor. Todos os pings tiveram sucesso, confirmando que o roteamento entre VLANs e o DHCP Relay estão funcionando de ponta a ponta.

<p align="center">
<img src="images/12-teste-conectividade-final.png" width="1000">
</p>

---

## Conclusão

Esse laboratório mostrou na prática o porquê de o DHCP não funcionar sozinho em uma rede com várias VLANs: sem o ip helper-address, o broadcast da requisição não sai da rede local do cliente, então nunca chega no servidor que está em outra sub-rede.
Configurar o trunk nos switches e as subinterfaces com encapsulation dot1Q no roteador (router-on-a-stick) ajudou a fixar como essas duas camadas trabalham juntas pra separar e depois rotear entre as VLANs.
O ponto mais importante foi o troubleshooting da VLAN 30. Comparei a configuração dela com as VLANs que já estavam funcionando e usei o show ip interface pra investigar. Foi assim que descobri que faltava o helper e resolvi.

## Arquivos do laboratório

Caso queira reproduzir a configuração, o arquivo do Cisco Packet Tracer está disponível para download neste diretório.

- [lab-dhcp-relay-vlans.pkt](./lab-dhcp-relay-vlans.pkt)

 
## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação

Focado em Infraestrutura, Redes de Computadores e GNU/Linux.

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
