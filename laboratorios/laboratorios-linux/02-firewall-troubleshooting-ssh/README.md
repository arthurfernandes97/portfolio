<h1 align="center">Gerenciamento de Firewall com firewalld (CentOS e Debian)</h1>

Nesse laboratório usei o `firewalld` para controlar o acesso SSH num servidor CentOS. Removi e restaurei a permissão do serviço SSH para ver na prática como o firewall interfere nas conexões remotas vindas de um cliente Debian.

## Tecnologias utilizadas
- **Virtualização:** Oracle VirtualBox
- **Sistemas Operacionais:** CentOS (Servidor) e Debian (Cliente)
- **Ferramentas:** SSH (Secure Shell), firewalld

## Cenário
Usei duas VMs ligadas por uma rede interna do VirtualBox, isoladas da minha rede real. Debian como cliente, CentOS como servidor, endereços da faixa 10.0.0.0/24.

## 1. Preparação
Configurei o IP das duas máquinas para elas se enxergarem:
- **Debian (Cliente):** 10.0.0.2
- **CentOS (Servidor):** 10.0.0.3

<p align="center">
<img src="./imagens/01-configurando-ip.png" width="1000">
</p>

Testei a conectividade com ping.

<p align="center">
<img src="./imagens/02-testando-conectividade.png" width="1000">
</p>

Criei uma pasta (`Pasta-CentOS`) no CentOS para confirmar depois, na conexão SSH, que eu realmente estava acessando aquela máquina e não outra.

<p align="center">
<img src="./imagens/03-criacao-diretorio-centos.png" width="600">
</p>

---

## 2. Acesso SSH
Com o ambiente pronto, testei o SSH do Debian pro CentOS antes de mexer em qualquer regra de firewall. Funcionou de primeira, e o `ls` mostrou a pasta que eu tinha criado, confirmando que estava no lugar certo.

<p align="center">
<img src="./imagens/04-teste-ssh-inicial.png" width="600">
</p>

---

## 3. Bloqueando o SSH no firewall
Rodei um `firewall-cmd --list-all` e vi que o serviço SSH estava liberado. Removi com:

```
firewall-cmd --remove-service=ssh --permanent
firewall-cmd --reload
```

<p align="center">
<img src="./imagens/05-remocao-ssh-firewall.png" width="800">
</p>

---

## 4. Testando o bloqueio
Tentei conectar novamente pelo Debian. Dessa vez não conectou:
```
ssh: connect to host 10.0.0.3 port 22: No route to host
```

<p align="center">
<img src="./imagens/06-erro-conexao.png" width="600">
</p>

---

## 5. Revertendo
Voltei o serviço no firewall:
```
firewall-cmd --add-service=ssh --permanent
firewall-cmd --reload
```

<p align="center">
<img src="./imagens/07-reversao-ssh.png" width="600">
</p>

---

## 6. Confirmando que voltou
Testei o SSH novamente no Debian, e dessa vez conectou normalmente.

<p align="center">
<img src="./imagens/08-teste-ssh-final.png" width="600">
</p>

---

## Conclusão
Esse lab deixou claro que o `firewalld` controla direto quais serviços conseguem receber conexão de fora. Com o SSH liberado, a conexão funciona. Sem essa permissão, ela passa a retornar "No route to host", mesmo com o resto da rede configurado corretamente.

Um detalhe que vale registrar. No meio do processo, tentei remover o serviço usando SSH maiúsculo (`--remove-service=SSH`), e o comando "funcionou" sem erro, mas na verdade não removeu nada. O `firewalld` diferencia maiúsculo de minúsculo nos nomes de serviço e o nome real é `ssh`. Só percebi o problema porque a conexão continuou funcionando depois do "bloqueio", o que não devia acontecer.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
