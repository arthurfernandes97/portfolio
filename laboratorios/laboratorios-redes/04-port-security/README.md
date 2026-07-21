<h1 align="center">Port Security</h1>

## Objetivo
Nesse laboratório reaproveitei a topologia do lab [ACL entre VLANs: Restrição de Acesso ao Servidor](https://github.com/arthurfernandes97/portfolio/tree/main/laboratorios/laboratorios-redes/03-acl-restricao-servidor), dando continuidade na mesma estrutura de rede para implementar `Port Security` nos switches.

A ideia foi impedir que outros dispositivos, diferentes dos que já estavam conectados, conseguissem acesso à rede pelas portas configuradas com `Port Security`.

## Tecnologias utilizadas

- Cisco Packet Tracer
- VLANs
- Port Security
- Sticky MAC Address
- ICMP (Ping)


## Topologia

Reutilizei a topologia criada nos laboratórios anteriores, onde já estavam configurados VLANs, DHCP Relay e ACLs.

- **VLAN 10** - Vendas
- **VLAN 20** - Administração
- **VLAN 30** - TI
- **VLAN 40** - Servidor

<p align="center">
<img src="imagens/01-topologia.png" width="850">
</p>

---

## Teste inicial (baseline)
Antes de aplicar qualquer configuração, testei a conectividade das três VLANs. Todas responderam normalmente, como esperado.

Vendas e Administração continuam sem acesso à VLAN 40 (`172.16.40.2`), por causa da ACL configurada no laboratório anterior.

<p align="center">
<img src="imagens/02-baseline.png" width="1000">
</p>

---

## Configurando Port Security
Ativei o Port Security nas portas de acesso dos dois switches, limitando a 1 MAC por porta, com aprendizado dinâmico (`sticky`) e violação configurada para derrubar a porta (`shutdown`).

**Switch 1:**

<p align="center">
<img src="imagens/03-port-security-switch1.png" width="1000">
</p>

**Switch 2:**

<p align="center">
<img src="imagens/04-port-security-switch2.png" width="1000">
</p>

---

## Aprendizado dos endereços MAC
Logo depois de configurar, a tabela de MACs seguros estava vazia nos dois switches. Isso porque o modo `sticky` aprende o MAC de forma dinâmica, só quando passa tráfego de verdade pela porta, não no momento em que o comando é digitado.

<p align="center">
<img src="imagens/05-nenhum-mac-aprendido.png" width="1000">
</p>

Depois de dar ping entre os dispositivos, a tabela passou a mostrar o MAC de cada porta, confirmando que o aprendizado dinâmico funcionou.

<p align="center">
<img src="imagens/06-mac-aprendido.png" width="1000">
</p>

---

## Testando o bloqueio (Switch 1)
Para testar a violação, desconectei o Laptop2 da porta Fa0/5 e conectei um PC de teste no lugar.

O DHCP não funcionou para o PC de teste, e no início eu achei que fosse algum problema à parte. Depois de analisar o comportamento, o motivo é o próprio Port Security: como o MAC do PC de teste é diferente do que já estava salvo naquela porta, o switch derruba a porta assim que detecta o MAC errado, mesmo antes do processo de DHCP conseguir terminar. Por isso configurei IP estático, só para conseguir fazer o teste de ping.

<p align="center">
<img src="imagens/07-ip-pc-teste-switch1.png" width="850">
</p>

Tentei o ping do PC de teste para rede da VLAN 20:

<p align="center">
<img src="imagens/08-ping-pc-teste-switch1.png" width="750">
</p>

E confirmei a violação no log do switch, mostrando o MAC não autorizado (`0040.0B84.71E0`) e a porta indo para estado de erro (`err-disable`):

<p align="center">
<img src="imagens/09-violation-switch1.png" width="1000">
</p>

---

## Testando o bloqueio (Switch 2)
Repeti o mesmo processo no Switch 2, trocando um dos dispositivos autorizados pelo PC de teste.

<p align="center">
<img src="imagens/10-ip-pc-teste-switch2.png" width="850">
</p>

<p align="center">
<img src="imagens/11-ping-pc-teste-switch2.png" width="750">
</p>

<p align="center">
<img src="imagens/12-violation-switch2.png" width="1000">
</p>

---

## Reestabelecendo a conexão
Depois de reconectar os dispositivos originais (com o MAC já autorizado na tabela), a porta continuava em estado de erro, mesmo o MAC batendo com o que já estava salvo como `sticky`.

<p align="center">
<img src="imagens/13-erro-conexao.png" width="850">
</p>

Fui estudar o porquê, e entendi que o Port Security não volta sozinho de um estado de violação, mesmo com o dispositivo certo reconectado. A porta fica em `err-disable` até alguém intervir manualmente. A correção foi entrar na interface, direto no switch, e alternar o estado dela:
```
Switch(config)#interface f0/5
Switch(config-if)#shutdown
Switch(config-if)#no shutdown
```
Repeti o mesmo processo na porta correspondente do Switch 2. Depois disso, as portas voltaram ao normal e a conectividade original foi restabelecida.

<p align="center">
<img src="imagens/14-reestabelecendo-conexao.png" width="1000">
</p>

---

## Conclusão
Esse lab me mostrou que Port Security trabalha numa camada diferente da ACL: enquanto a ACL controla o que pode trafegar entre redes (camada 3, por IP), o Port Security controla quem pode nem entrar na rede pela porta física do switch (camada 2, por MAC). Trocar um dispositivo autorizado por outro na mesma porta é o suficiente para derrubar a conexão inteira, mesmo antes de qualquer IP ser atribuído.

O ponto mais importante foi entender que a violação não se resolve sozinha. Mesmo reconectando o dispositivo certo, com o MAC batendo na tabela `sticky`, a porta continua bloqueada até alguém reiniciar ela manualmente com `shutdown` / `no shutdown`. Isso me mostrou que Port Security é uma ferramenta de contenção, não corrige o problema automaticamente, só impede o acesso até alguém agir.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
