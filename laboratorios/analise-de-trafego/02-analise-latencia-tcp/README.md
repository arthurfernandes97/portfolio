<h1 align="center">Análise de Latência TCP: Delayed ACK em uma Captura NFS</h1>

## Objetivo
Esse laboratório tem como objetivo investigar picos de latência numa captura de tráfego real, identificando a causa raiz de atrasos aparentes numa conexão TCP.

## Metodologia
Usei um arquivo de captura de referência da comunidade Wireshark, focado em atrasos de rede.

- **Fonte:** https://wiki.wireshark.org/samplecaptures
- **Arquivo usado:** [nfs_bad_stalls.cap](./nfs_bad_stalls.cap), descrito na wiki como uma captura NFS com atrasos de 38ms no meio de várias respostas de leitura
  
`Arquivo disponível para download neste repositório.`

## Tecnologias utilizadas
- Wireshark
- Filtros de exibição (`tcp.time_delta`, `tcp.stream`)
- Análise de campos `[SEQ/ACK analysis]` (RTT e iRTT)

## Visão geral da captura
Abri o arquivo sem filtro nenhum pra ter uma ideia geral do tráfego. No total, **7038 pacotes**. Já dá pra ver logo no início: resolução via ARP, handshakes TCP, chamadas `Portmap` (que resolve a porta de um serviço RPC), `MOUNT` (montagem do compartilhamento) e depois as chamadas NFS propriamente ditas (`GETATTR`, `FSSTAT`, `FSINFO`, `ACCESS`).

<p align="center">
<img src="imagens/01-visao-geral-captura.png" width="850">
</p>

---

## Isolando os pacotes com atraso
Apliquei o filtro:
```
tcp.time_delta > 0.03
```
Isso mostra pacotes onde passou mais de 30 milissegundos (0.03s) desde a última resposta daquele mesmo stream TCP. O filtro retornou **50 pacotes** de 7038.

No começo entendi errado o que o filtro estava mostrando, achei que a diferença de tempo era entre um resultado da lista filtrada e o próximo (por exemplo, do pacote 224 pro 400). Não é isso. Cada linha do filtro mostra o atraso daquele pacote **em relação ao pacote imediatamente anterior do mesmo stream**, no arquivo inteiro, não em relação ao resultado anterior da lista filtrada.

<p align="center">
<img src="imagens/02-filtro-tcp-time-delta.png" width="850">
</p>

---

## Investigando um dos pacotes
Voltei a ver a captura sem filtro, ao redor do pacote 400, e reparei que o pacote 399 (um `RPC Continuation`) chegou em `4.103510`, e o pacote 400 (um `TCP ACK`) só apareceu em `4.140639`, uma diferença de 37ms.

Pra ver o stream inteiro sem misturar com outros fluxos, usei:
```
tcp.stream eq 2
```
Isso mostrou uma rajada de fragmentos `RPC Continuation` sendo confirmados quase instantaneamente um atrás do outro, exceto o último ACK da rajada (pacote 400), que demorou visivelmente mais.

<p align="center">
<img src="imagens/03-stream-isolado.png" width="850">
</p>

---

## Confirmando a causa com o SEQ/ACK analysis
Abrindo os detalhes do pacote 400, no campo `[SEQ/ACK analysis]`:
```
[This is an ACK to the segment in frame: 399]
[The RTT to ACK the segment was: 37.129000 milliseconds]
[iRTT: 98.000 microseconds]
```
O `iRTT` é o RTT inicial da conexão, medido lá no handshake. Nesse caso, **0.098ms**, extremamente rápido, típico de rede local. Comparado com os 37ms que esse ACK específico demorou, fica claro que não é a rede que está lenta: a rede responde rápido, só esse ACK em particular demorou a ser enviado.

<p align="center">
<img src="imagens/04-seq-ack-analysis-pacote-400.png" width="850">
</p>

---

## Confirmando o padrão em um segundo pacote
Pra não tirar conclusão em cima de um único exemplo, testei outro pacote da lista filtrada (o pacote 1022):
```
[This is an ACK to the segment in frame: 1021]
[The RTT to ACK the segment was: 38.084000 milliseconds]
[iRTT: 98.000 microseconds]
```
O `iRTT` é o mesmo (mesma conexão), e o RTT do ACK ficou bem próximo do primeiro caso (38ms contra 37ms). Valores tão parecidos entre dois eventos diferentes não parecem coincidência de rede, parecem um comportamento fixo repetido.

<p align="center">
<img src="imagens/05-confirmacao-pacote-1022.png" width="850">
</p>

---

## O que é isso, de fato
O padrão bate com o comportamento conhecido como **TCP Delayed ACK**, inclusive eu ainda não tinha o conhecimento desse padrão de comportamento. Ao pesquisar sobre o tema, eu aprendi que em vez de confirmar cada pacote de dado recebido imediatamente, alguns sistemas esperam um pequeno intervalo (por padrão, próximo de 40ms em muitas implementações) antes de mandar o ACK na esperança de confirmar vários pacotes de uma vez só, ou aproveitar esse mesmo pacote pra já mandar dado de volta junto (piggybacking). Isso reduz a quantidade de tráfego "administrativo" na rede, em troca de um pequeno atraso na confirmação.

O dado em si (pacote 399) já tinha chegado rápido. Só a confirmação formal (o ACK) que esperou de propósito. Não é um defeito de rede nem do servidor, é uma otimização normal do protocolo TCP.

## Conclusão
Comecei esse lab suspeitando de rede lenta ou servidor sobrecarregado, mas ao comparar o RTT do ACK com o iRTT da conexão, ficou claro que a rede em si era rápida. Isso me levou a entender que o atraso era um comportamento de Delayed ACK do TCP, não um problema real de desempenho. Confirmei isso testando um segundo pacote e vendo o mesmo padrão se repetir.

Vale registrar: os conceitos de RTT, iRTT e Delayed ACK eu não conhecia antes desse lab. Entendi estudando enquanto analisava essa captura, com apoio de material e explicações que fui buscando conforme surgiam as dúvidas, principalmente por que 0.03s era um valor alto pra rede local, e por que a demora estava no ACK e não no dado em si. O que não veio de fora foi o processo: errar a leitura do filtro de primeira, voltar pra reanalisar sem filtro, e testar num segundo pacote antes de aceitar a explicação.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
