<h1 align="center">Configuração de Estação Windows 11 para Novo Colaborador</h1>

## Objetivo

Simular a preparação de uma estação de trabalho Windows 11 para um novo colaborador do setor de TI, desde a criação da máquina virtual até a entrega do ambiente pronto para uso. O laboratório inclui a instalação do sistema operacional, configuração inicial, validação da conectividade de rede, integração entre host e máquina virtual e instalação de aplicativos essenciais.

## Tecnologias utilizadas

- Windows 11 Home (pt-BR)
- Oracle VirtualBox
- VirtualBox Guest Additions
- Prompt de Comando (`ipconfig`, `ping` e `nslookup`)
- Google Chrome
- Visual Studio Code
- 7-Zip

---

## Etapa 1 - Criação da Máquina Virtual

Criei a máquina virtual no Oracle VirtualBox com o nome **WKS-TI-01**, usando a ISO oficial do Windows 11. Configurei **4 GB de memória RAM**, **2 CPUs** e um **disco virtual dinâmico de 80 GB (VDI)**. Também habilitei **EFI**, **TPM 2.0** e **Secure Boot**, os três requisitos que o Windows 11 exige para instalar.

<p align="center"><img src="imagens/01-vm-nome-iso.png" width="850"></p>

<p align="center"><img src="imagens/02-vm-hardware-efi.png" width="850"></p>

<p align="center"><img src="imagens/03-vm-disco-virtual.png" width="850"></p>

<p align="center"><img src="imagens/04-vm-tpm-uefi-secureboot.png" width="850"></p>

---

## Etapa 2 - Instalação do Windows 11

Após iniciar a máquina virtual, acessei manualmente o menu de boot da UEFI para selecionar a mídia de instalação do Windows 11. Em seguida, utilizei o assistente padrão da Microsoft para definir o idioma, prosseguir sem informar uma chave de produto (ambiente de laboratório), selecionar a edição **Windows 11 Home** e permitir que o instalador criasse automaticamente as partições do disco virtual.

Após a cópia dos arquivos e o primeiro reinício, o dispositivo foi identificado como **WKS-TI-01**.

<p align="center"><img src="imagens/05-boot-manager-menu.png" width="850"></p>

<p align="center"><img src="imagens/06-boot-selecao-uefi-cdrom.png" width="850"></p>

<p align="center"><img src="imagens/07-instalacao-idioma.png" width="850"></p>

<p align="center"><img src="imagens/08-instalacao-sem-chave-produto.png" width="850"></p>

<p align="center"><img src="imagens/09-instalacao-particionamento.png" width="850"></p>

<p align="center"><img src="imagens/10-instalacao-progresso.png" width="850"></p>

<p align="center"><img src="imagens/11-instalacao-nome-dispositivo.png" width="850"></p>

---

## Etapa 3 - Configuração Inicial

Durante a configuração inicial, o Windows tentou exigir o uso de uma conta Microsoft antes de liberar o acesso ao sistema. Como o objetivo do laboratório era utilizar uma conta local, tentei utilizar o comando `oobe\bypassnro` pelo Prompt de Comando (`Shift + F10`).

Nesta versão do Windows 11, o comando não foi suficiente para liberar a configuração offline. A solução foi desabilitar temporariamente o adaptador de rede da máquina virtual (**VirtualBox → Configurações → Rede → Habilitar Placa de Rede**), permitindo que o Windows exibisse a opção de criação de uma conta local.

<p align="center"><img src="imagens/12-troubleshooting-rede-desabilitada.png" width="850"></p>

Com a rede temporariamente desabilitada, consegui concluir a configuração inicial utilizando o usuário local **Colaborador** e acessar a área de trabalho do Windows. Em seguida, o adaptador de rede foi reabilitado para dar continuidade às próximas etapas do laboratório.

<p align="center"><img src="imagens/13-instalacao-usuario-local.png" width="850"></p>

<p align="center"><img src="imagens/14-windows-primeiro-login.png" width="850"></p>

---

## Etapa 4 - Validação da Conectividade de Rede

Com a instalação concluída e o adaptador de rede novamente habilitado, realizei alguns testes para confirmar que a estação havia recebido um endereço IP automaticamente e possuía acesso à Internet.

A configuração foi validada utilizando o comando `ipconfig /all`, enquanto a conectividade e a resolução de nomes foram verificadas com os comandos `ping` e `nslookup`.

<p align="center"><img src="imagens/15-ipconfig.png" width="850"></p>

<p align="center"><img src="imagens/16-teste-conectividade.png" width="850"></p>

---

## Etapa 5 - Integração entre Host e Máquina Virtual

Após concluir a instalação do Windows 11, instalei o **VirtualBox Guest Additions**, adicionando recursos que facilitam o uso da máquina virtual, como o redimensionamento automático da tela, a área de transferência compartilhada e o suporte a pastas compartilhadas.

<p align="center"><img src="imagens/17-guest-additions.png" width="850"></p>

Em seguida, criei uma pasta compartilhada no computador hospedeiro e configurei no Oracle VirtualBox para montagem automática na máquina virtual. Após a configuração, a pasta passou a ficar disponível como uma unidade de rede no Windows, permitindo a troca de arquivos entre o host e a VM.

<p align="center"><img src="imagens/19-criando-pasta-host.png" width="850"></p>

<p align="center"><img src="imagens/20-compartilhando-pasta-vm.png" width="850"></p>

<p align="center"><img src="imagens/21-pasta-compartilhada.png" width="850"></p>

---

## Etapa 6 - Instalação de Aplicativos e Windows Update

Após concluir as configurações iniciais da estação, instalei os aplicativos definidos para o ambiente de trabalho: **Google Chrome**, **Visual Studio Code** e **7-Zip**.

<p align="center"><img src="imagens/22-aplicativos-instalados.png" width="850"></p>

<p align="center"><img src="imagens/23-confirmacao-vscode.png" width="850"></p>

<p align="center"><img src="imagens/24-confirmacao-chrome.png" width="850"></p>

Na sequência, executei o **Windows Update** até que todas as atualizações disponíveis fossem instaladas.

<p align="center"><img src="imagens/18-windows-update.png" width="850"></p>

---

## Etapa 7 - Snapshot do Estado Final

Para preservar a configuração da estação, criei um snapshot da máquina virtual chamado **Estado-inicial-pronto-para-entrega**. Esse ponto de restauração permite retornar rapidamente ao ambiente configurado caso seja necessário realizar novos testes ou reverter alterações futuras.

<p align="center"><img src="imagens/25-snapshot.png" width="850"></p>

> **Nota:** O snapshot é um recurso específico de ambientes virtualizados e não faz parte da entrega de uma estação física ao usuário. Em plataformas de virtualização, ele é amplamente utilizado para preservar um estado conhecido da máquina e facilitar sua recuperação quando necessário.

---

## Conclusão

Fechando esse laboratório, a `WKS-TI-01` ficou pronta como estação de trabalho. Windows 11 instalado, rede validada, Guest Additions e pasta compartilhada funcionando, os três aplicativos instalados e o Windows Update em dia.

O ponto que mais me ensinou foi o bypass da conta Microsoft. Não bastou só rodar o `oobe\bypassnro`, precisei também desabilitar o adaptador de rede da VM para forçar o Windows a liberar a opção de conta local. Isso mostrou que a Microsoft vem fechando esses atalhos de versão em versão, e que às vezes a solução não está no comando "oficial" que todo mundo indica, e sim em contornar o requisito por outro caminho. Nesse caso, tirar a condição de internet disponível foi o que fez o Windows parar de insistir na conta online.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)