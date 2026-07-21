# Laboratório Linux - Diretórios Compartilhados com SGID e Sticky Bit

Laboratório sobre permissões especiais em diretórios compartilhados. O foco foi entender duas permissões que eu não tinha usado na prática ainda: SGID (herança automática de grupo) e Sticky Bit (proteção contra exclusão de arquivo por outro usuário do mesmo grupo).

## Tecnologias Utilizadas

- **Virtualização:** Oracle VirtualBox
- **Sistemas Operacionais:** Debian
- **Ferramentas:** GNU/Linux, Bash

## Cenário

Um diretório compartilhado entre usuários do mesmo grupo (`equipe`). A ideia era simular uma situação comum em ambientes corporativos: vários usuários com acesso ao mesmo diretório, mas sem controle automático de grupo nos arquivos criados e sem proteção contra exclusão cruzada.

Antes de qualquer configuração especial, dois problemas aparecem:
- arquivos criados dentro do diretório não herdam o grupo do diretório, e sim o grupo pessoal de quem criou;
- qualquer usuário com permissão de escrita no diretório pode apagar arquivos de outro usuário.

## Ambiente

- Distribuição: Debian (VM local, VirtualBox)
- Usuários de teste: `joao`, `jose`
- Grupo de teste: `equipe`

## Preparação

Criei o grupo e os dois usuários de teste, e adicionei ambos ao grupo `equipe`:

```bash
sudo groupadd equipe
sudo useradd joao
sudo useradd jose
sudo usermod -aG equipe joao
sudo usermod -aG equipe jose
```

Conferi se os dois realmente ficaram no grupo:

```bash
groups joao
# joao : joao equipe

groups jose
# jose : jose equipe
```

Depois criei o diretório compartilhado, ajustei o grupo dono e apliquei a permissão base:

```bash
mkdir /home/compartilhado
chown root:equipe /home/compartilhado
chmod 770 /home/compartilhado
```

```
drwxrwx--- 2 root equipe 4096 Jul  7 16:44 compartilhado
```

<p align="center">
<img src="imagens/01-preparacao.png" width="850">
</p>

---

## Parte 1 - Sem SGID (comportamento padrão)

Logado como `joao`, criei um arquivo dentro do diretório compartilhado pra ver o que acontecia sem nenhuma permissão especial:

```bash
touch teste-sem-sgid.txt
ls -l
```

```
-rw-rw-r-- 1 joao joao 0 Jul  7 17:02 teste-sem-sgid.txt
```

O grupo do arquivo veio como `joao`, não `equipe`. Faz sentido — sem SGID, o arquivo herda o grupo primário de quem criou, não o grupo do diretório. Esse é justamente o problema que o laboratório se propõe a resolver.

<p align="center">
<img src="imagens/02-sem-sgid.png" width="850">
</p>

---

## Parte 2 - Aplicando SGID

Voltei pra root e apliquei o SGID no diretório:

```bash
chmod g+s /home/compartilhado
ls -l /home
```

```
drwxrws--- 2 root equipe 4096 Jul  7 17:02 compartilhado
```

O `s` no lugar do `x` da permissão de grupo confirma que o SGID está ativo.

<p align="center">
<img src="imagens/03-aplicando-sgid.png" width="850">
</p>

---

Repeti o teste, de novo como `joao`:

```bash
touch teste-com-sgid.txt
ls -l
```

```
-rw-rw-r-- 1 joao equipe 0 Jul  7 17:09 teste-com-sgid.txt
-rw-rw-r-- 1 joao joao   0 Jul  7 17:02 teste-sem-sgid.txt
```

Dessa vez o grupo do arquivo saiu `equipe`, herdado do diretório. Dá pra ver bem a diferença comparando as duas linhas — mesmo usuário (`joao`), mas grupo diferente dependendo de se o SGID estava ativo ou não na hora da criação.

<p align="center">
<img src="imagens/04-comparando.png" width="850">
</p>

---

## Parte 3 - Sticky Bit

Ainda como root, apliquei o Sticky Bit em cima do que já estava configurado:

```bash
chmod +t /home/compartilhado
ls -l /home
```

```
drwxrws--T 2 root equipe 4096 Jul  7 17:09 compartilhado
```

Reparei que apareceu o `T` maiúsculo, não o `t` minúsculo — isso acontece porque a permissão de execução pra "outros" está em `0` (base `770`), então não tem `x` pra combinar com o Sticky Bit.

Pra testar, entrei como `jose` e tentei apagar um arquivo que era do `joao`:

```bash
rm teste-com-sgid.txt
# rm: cannot remove 'teste-com-sgid.txt': Operation not permitted

rm -rf teste-com-sgid.txt
# rm: cannot remove 'teste-com-sgid.txt': Operation not permitted
```

Bloqueou nas duas tentativas, mesmo com o `-f`. Isso confirma que a restrição é do sistema de arquivos mesmo, não é só uma confirmação que o `rm` pede e o `-f` pula.

<p align="center">
<img src="imagens/05-sticky-bit.png" width="850">
</p>

---

## Troubleshooting

Alguns problemas apareceram no meio do processo, principalmente na parte de preparação do ambiente:

Logo no começo, tentei usar `su` sem hífen pra virar root, e os comandos `useradd` e `usermod` deram `command not found`. Levei um tempo pra entender — o motivo é que `su` sem `-` não recarrega o `PATH` do root, então comandos que ficam em `/usr/sbin` não são encontrados. Resolvi usando `sudo` direto nos comandos de administração.

Também errei a sintaxe ao tentar trocar pro usuário `joao` — digitei `su -joao` sem espaço, e o shell interpretou como se eu estivesse passando uma opção inválida (`invalid option -- 'j'`). Bastava colocar o espaço entre o hífen e o nome do usuário.

Outra coisa que reparei: os usuários `joao` e `jose` foram criados sem a flag `-m` do `useradd`, então não têm diretório home. Ao trocar de usuário com `su - joao`, o sistema avisa que não conseguiu entrar em `/home/joao`, mas segue o login normalmente, caindo em `/home`. Não atrapalhou o teste porque o diretório compartilhado também está em `/home`, mas é algo pra corrigir se algum dia esses usuários precisarem de home própria.

Por fim, tentei rodar `chown equipe compartilhado/` esperando que isso mudasse o grupo — deu erro de `invalid user: 'equipe'`, porque `chown` sozinho espera um usuário, não um grupo. A forma certa é `chown root:equipe compartilhado/`, que define dono e grupo de uma vez.

## Permissões finais aplicadas

| Permissão | Valor | Efeito |
|---|---|---|
| Base | `770` | Dono e grupo com leitura/escrita/execução; outros sem acesso |
| SGID | `2770` | Arquivos criados dentro do diretório herdam o grupo `equipe` |
| Sticky Bit | `3770` | Cada usuário só pode apagar os próprios arquivos, mesmo com escrita liberada para o grupo |

## Aprendizados

Esse laboratório deixou bem claro por que SGID e Sticky Bit existem separados do `chmod` comum. Sem o SGID, seria necessário rodar `chgrp` manualmente toda vez que alguém criasse um arquivo no diretório — algo fácil de esquecer, especialmente com vários usuários usando o mesmo espaço. E sem o Sticky Bit, ter escrita compartilhada no grupo significa, na prática, que qualquer um pode apagar o que quiser dentro do diretório, não só o que criou.

A parte de troubleshooting acabou rendendo mais aprendizado do que eu esperava. Os erros de PATH e de sintaxe do `su` não tinham nada a ver com SGID ou Sticky Bit diretamente, mas mostraram detalhes de como o Linux lida com sessões e variáveis de ambiente que eu não tinha parado pra entender antes.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
