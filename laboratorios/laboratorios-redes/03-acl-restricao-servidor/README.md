<h1 align="center">ACL entre VLANs: Restrição de Acesso ao Servidor</h1>

## Objetivo
Neste laboratório reaproveitei a topologia do lab de [DHCP Relay com Múltiplas VLANs](https://github.com/arthurfernandes97/portfolio/tree/main/laboratorios/laboratorios-redes/02-dhcp-relay-vlans), dando continuidade na mesma estrutura de rede para implementar controle de acesso entre as VLANs.

A ideia foi impedir que Vendas e Administração acessassem a VLAN do servidor usando ACLs estendidas, mantendo esse acesso liberado somente para TI. No fim, também verifiquei se a comunicação entre as outras VLANs e o funcionamento do DHCP Relay continuavam normais depois da aplicação das regras.

## Tecnologias utilizadas
- Cisco Packet Tracer
- VLANs, Router-on-a-Stick
- ACLs Estendidas
- DHCP Relay (`ip helper-address`)
- ICMP (Ping)

## Regra de acesso
- Vendas não pode acessar o servidor
- Administração não pode acessar o servidor
- TI continua com acesso liberado ao servidor
- Vendas e Administração continuam se comunicando normalmente entre si

## Topologia
Utilizei a mesma topologia do laboratório anterior:
- **VLAN 10** - Vendas
- **VLAN 20** - Administração
- **VLAN 30** - TI
- **VLAN 40** - Servidor

<p align="center">
<img src="imagens/01-topologia.png" width="850">
</p>

---

## Teste inicial (baseline)
Antes de aplicar qualquer ACL, testei a conectividade das três VLANs de cliente com o servidor. Todas responderam normalmente.

<p align="center">
<img src="imagens/02-teste-baseline.png" width="1000">
</p>

---

## Configuração das ACLs
Criei duas ACLs estendidas. A **110** bloqueia Vendas, a **120** bloqueia Administração:
```
access-list 110 deny ip 172.16.10.0 0.0.0.255 172.16.40.0 0.0.0.255
access-list 110 permit ip any any

access-list 120 deny ip 172.16.20.0 0.0.0.255 172.16.40.0 0.0.0.255
access-list 120 permit ip any any
```
A linha `permit ip any any` é obrigatória em cada uma. Toda ACL tem um "deny all" implícito no final, então sem essa linha, a VLAN inteira perderia acesso a tudo, não só ao servidor.

Apliquei cada ACL na subinterface correspondente. Vendas e Administração estão na mesma interface física do roteador:
```
interface g0/0/0.10
 ip access-group 110 in

interface g0/0/0.20
 ip access-group 120 in
```

<p align="center">
<img src="imagens/03-criando-acl.png" width="600">
</p>

---

## Testando o bloqueio
Depois da configuração, repeti os testes de conectividade. Vendas e Administração deixaram de acessar o servidor, enquanto TI continuou acessando normalmente.

<p align="center">
<img src="imagens/04-testando-bloqueio-acl.png" width="1000">
</p>

---

## Comunicação entre Vendas e Administração
Testei também se Vendas e Administração ainda se enxergavam entre si. Continuou funcionando normalmente nos dois sentidos, confirmando que a ACL bloqueou só o acesso ao servidor, não a VLAN inteira.

<p align="center">
<img src="imagens/05-testes-geral-ping.png" width="850">
</p>

---

## Verificando o DHCP
Como essa topologia depende de DHCP Relay, eu esperava que a ACL também bloqueasse isso, já que a regra usa `deny ip` de forma genérica, sem distinguir protocolo nem porta. Fiz o teste de qualquer forma: `ipconfig /release` seguido de `ipconfig /renew` nos clientes de Vendas e Administração.

<p align="center">
<img src="imagens/06-ip-release-renew.png" width="1000">
</p>

Os dois pegaram IP novo normalmente, sem nenhum problema. Isso me deixou curioso, porque não era o que eu esperava.

Fui entender o porquê: quando o cliente manda a requisição DHCP, o pacote sai como broadcast (endereçado a `255.255.255.255`, não a um IP específico). A checagem da ACL de entrada acontece nesse momento, contra esse destino de broadcast, que não bate com a condição da regra (`172.16.40.0/24`). Só depois disso o roteador, através do `ip helper-address`, reescreve o pacote como unicast, endereçado direto ao servidor. Ou seja, a ACL nunca chega a avaliar o pacote já reescrito com o IP do servidor como destino. Por isso o bloqueio não afeta o DHCP, mesmo sendo uma regra genérica.

## Verificando as ACLs
Tentei confirmar as regras com `show access-list`, mas recebi um erro:
```
Router(config)#show access-list
                    ^
% Invalid input detected at '^' marker.
```
Mesmo erro que já tinha visto no lab de DHCP Relay: comando de modo EXEC não roda direto dentro do modo `config`, precisa do prefixo `do`. Corrigi com:
```
Router(config)#do show access-list
```
E consegui ver as duas ACLs aplicadas, com a contagem de matches confirmando que as regras estavam sendo usadas de verdade:
```
Extended IP access list 110
    10 deny ip 172.16.10.0 0.0.0.255 172.16.40.0 0.0.0.255 (12 match(es))
    20 permit ip any any (31 match(es))
Extended IP access list 120
    10 deny ip 172.16.20.0 0.0.0.255 172.16.40.0 0.0.0.255 (4 match(es))
    20 permit ip any any (18 match(es))
```

<p align="center">
<img src="imagens/07-show-access-list.png" width="600">
</p>

---

## Conclusão
Esse lab mostrou como usar ACL estendida para restringir acesso entre redes específicas, sem bloquear tudo. Vendas e Administração ficaram sem acesso ao servidor, TI manteve acesso total, e as duas VLANs bloqueadas continuaram se comunicando entre si normalmente.

O ponto mais interessante foi o DHCP não quebrar, mesmo eu esperando que quebrasse. Entender que a ACL avalia o pacote de broadcast antes do `ip helper-address` reescrever ele pra unicast me ajudou a enxergar melhor a ordem em que essas coisas acontecem dentro do roteador.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)

