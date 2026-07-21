<h1 align="center">Análise de Navegação HTTP com Wireshark</h1>

## Objetivo
Nesse laboratório tive como objetivo analisar uma captura de navegação HTTP, acompanhando cada etapa da conexão: Three-Way Handshake, resolução DNS,  requisição, resposta e encerramento.

## Metodologia
Usei um arquivo de captura de referência da comunidade Wireshark.

- **Fonte:** https://wiki.wireshark.org/samplecaptures
- **Arquivo usado:** [`http.cap`](./http.cap)
 
`Arquivo disponível para download neste repositório.`

## Tecnologias utilizadas

- Wireshark
- HTTP
- TCP
- DNS
  
## Visão geral da captura
Abri o arquivo sem filtro nenhum para ter uma visão geral do tráfego antes de começar a análise.

<p align="center">
<img src="imagens/01-captura-http.png" width="850">
</p>

---

## Three-Way Handshake
Os primeiros três pacotes da captura são o handshake com o servidor na porta 80:
- **SYN:** o cliente pede para abrir a conexão.
- **SYN, ACK:** o servidor confirma e também quer conectar.
- **ACK:** o cliente confirma, e a conexão fica estabelecida.

O domínio `www.ethereal.com` já estava resolvido antes do início da captura, por isso o handshake aparece logo no começo, sem consulta DNS antes dele.

<p align="center">
<img src="imagens/02-three-way-handshake.png" width="850">
</p>

---

## Requisição HTTP GET
Com a conexão estabelecida, o cliente faz a requisição, no pacote 4:
```
GET /download.html HTTP/1.1
Host: www.ethereal.com
```

<p align="center">
<img src="imagens/03-http-get.png" width="850">
</p>

---

## Resolução DNS de um recurso secundário
Nos pacotes 13 e 17, aparece uma consulta DNS, mas não é para o domínio principal. É para `pagead2.googlesyndication.com`, um script de anúncio referenciado dentro da própria página que estava sendo carregada. Só depois que o navegador processa parte do conteúdo de `download.html` é que ele descobre que precisa buscar esse recurso externo, e por isso essa consulta aparece bem depois do GET, não antes do handshake.

**Consulta (pacote 13):**

<p align="center">
<img src="imagens/04-dns-query.png" width="850">
</p>

**Resposta (pacote 17):**

<p align="center">
<img src="imagens/05-dns-response.png" width="850">
</p>

---

## Retransmissão suspeita e ACK duplicado
Mais adiante na captura, reparei que dois pacotes (36 e 37) apareciam destacados em preto na lista, diferente do resto. Fui investigar e descobri que o Wireshark sinaliza automaticamente esse tipo de comportamento. Esses dois pacotes pertencem à conexão com o servidor de anúncio resolvido no passo anterior.

No painel de detalhes do pacote 36, aparece:
```
[This frame is a (suspected) spurious retransmission]
```
O Wireshark suspeita que o servidor reenviou um dado que já tinha sido entregue antes.

<p align="center">
<img src="imagens/06-retransmissao-suspeita.png" width="850">
</p>

No pacote 37, o cliente confirma isso:
```
[This is a TCP duplicate ack]
[Duplicate ACK #: 1]
[Duplicate to the ACK in frame: 28]
```
Esse ACK é uma repetição do que já tinha sido confirmado lá no pacote 28.

<p align="center">
<img src="imagens/07-ack-duplicado.png" width="850">
</p>

Não sabia o que esses termos significavam antes de reparar nesse destaque. Fui atrás de entender, e percebi que os dois pacotes contam a mesma história: o servidor mandou algo de novo sem necessidade, e o cliente só confirmou de volta o que já tinha recebido antes.

## Resposta HTTP 200 OK
No pacote 38, chega a resposta completa da página principal, com status 200 OK.

<p align="center">
<img src="imagens/08-http-200-ok.png" width="850">
</p>

---

## Encerramento da conexão
Depois da troca de dados, a conexão é encerrada com FIN/ACK dos dois lados.

<p align="center">
<img src="imagens/09-finalizacao.png" width="850">
</p>

---

## Conclusão
Esse lab me ajudou a acompanhar o fluxo completo de uma navegação HTTP, do handshake até o encerramento da conexão. Como a comunicação utiliza HTTP, sem criptografia, foi possível ver em texto claro a requisição enviada pelo cliente e a resposta do servidor.

Uma coisa que não esperava foi que o DNS não aparecesse antes do handshake, como eu imaginava de início. Reparar que a consulta era pra um recurso secundário (um script de anúncio dentro da página), não pro domínio principal, me ajudou a entender melhor como uma página carrega várias coisas de lugares diferentes, cada uma com seu próprio ciclo de DNS e conexão.

O outro ponto que mais me chamou atenção foi a retransmissão suspeita. Não esperava encontrar isso numa captura que parecia simples, e foi só reparando na cor diferente da linha que fui atrás de entender o que o Wireshark estava sinalizando.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
