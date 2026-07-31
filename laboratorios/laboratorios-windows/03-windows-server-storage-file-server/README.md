<h1 align="center">Storage Pools, File Server e Permissões no Active Directory</h1>

## Objetivo

Neste laboratório implementei um ambiente de armazenamento no `SRV-TI-01` utilizando Storage Spaces, compartilhamento de pastas, permissões NTFS baseadas em grupos do Active Directory e cotas de disco com FSRM.

É a continuação direta do [`02-organizacao-usuarios-grupos-active-directory`](../02-organizacao-usuarios-grupos-active-directory), onde já estavam configurados o domínio `arthurtech.local`, as OUs, os usuários e os grupos de segurança.

## Cenário

O domínio `arthurtech.local` agora precisa de um servidor de arquivos para centralizar o armazenamento da empresa. Além do compartilhamento das pastas por departamento, o ambiente também deve restringir o acesso aos dados mais sensíveis, como RH e Financeiro, utilizando grupos do Active Directory, permissões NTFS e cotas de armazenamento.

## Tecnologias utilizadas

- Windows Server 2025
- Storage Spaces (Storage Pools, discos virtuais)
- Windows PowerShell (diagnóstico)
- Compartilhamento SMB e permissões NTFS
- Active Directory (grupos de segurança)
- FSRM (File Server Resource Manager)

---

## Etapa 1 - Arquitetura de armazenamento

Em vez de criar um Storage Pool por departamento, optei por separar o armazenamento conforme a sensibilidade dos dados:

- **Pool-Confidencial:** pastas `RH-Sigiloso` e `Financeiro`, com acesso restrito.
- **Pool-Operacional:** pastas `TI`, `Comercial`, `Administracao` e `Publica`, utilizadas no dia a dia pelos demais setores.

---

## Etapa 2 - Criação dos discos e dos pools

Criei quatro discos virtuais VDI de 10 GB no `SRV-TI-01`, dois para cada Storage Pool.

<p align="center"><img src="./imagens/01-criacao-discos-virtuais.png" width="1000"></p>

Ao tentar criar os pools pela interface gráfica, o Server Manager só reconhecia 1 dos 4 discos disponíveis, mesmo o PowerShell confirmando os 4 com `CanPool = True`.

<p align="center"><img src="./imagens/02-discos-nao-inicializados.png" width="850"></p>
<p align="center"><img src="./imagens/03-powershell-canpool-confirmado.png" width="850"></p>
<p align="center"><img src="./imagens/04-server-manager-storage-pool-inconsistencia.png" width="650"></p>
<p align="center"><img src="./imagens/05-new-storage-pool-wizard-disco-unico.png" width="650"></p>

Investigando o problema, comparei o `DeviceId`, `UniqueId` e o `SerialNumber` dos discos:

```powershell
Get-PhysicalDisk | Select FriendlyName, DeviceId, UniqueId, SerialNumber
```

Percebi que os quatro discos possuíam o mesmo `UniqueId`, apesar de cada um ter um `SerialNumber` diferente.

<p align="center"><img src="./imagens/06-powershell-uniqueid-causa-raiz.png" width="850"></p>

**Observação:** o Server Manager não reconhecia corretamente todos os discos disponíveis. Após pesquisar alternativas para esse comportamento, encontrei uma forma de criar os Storage Pools utilizando o PowerShell. Apliquei os comandos abaixo para concluir essa etapa e os registrei aqui para documentar o procedimento utilizado neste laboratório.

```powershell
$disks = Get-PhysicalDisk -CanPool $true

New-StoragePool -FriendlyName "Pool-Confidencial" `
  -StorageSubSystemFriendlyName "Windows Storage on SRV-TI-01" `
  -PhysicalDisks ($disks | Where-Object DeviceId -in @("1","2"))

New-StoragePool -FriendlyName "Pool-Operacional" `
  -StorageSubSystemFriendlyName "Windows Storage on SRV-TI-01" `
  -PhysicalDisks ($disks | Where-Object DeviceId -in @("3","4"))
```

<p align="center"><img src="./imagens/07-powershell-storage-pools-criados.png" width="850"></p>

---

## Etapa 3 - Discos virtuais e volumes

Inicialmente pretendia utilizar o layout Mirror no `Pool-Confidencial`, mas essa opção não ficou disponível utilizando apenas dois discos virtuais de 10 GB.

Por esse motivo utilizei o layout Simple nos dois pools.

Em um ambiente real, adicionaria mais discos para implementar um volume em Mirror no `Pool-Confidencial`. 

<p align="center"><img src="./imagens/08-criacao-disco-virtual.png" width="1000"></p>

Em seguida criei os volumes, formatados em NTFS: `(E:)Confidencial` e `(F:)Operacional`.

<p align="center"><img src="./imagens/09-criacao-volume.png" width="1000"></p>
<p align="center"><img src="./imagens/10-volumes-criados-explorador-arquivos.png" width="650"></p>

---

## Etapa 4 - Estrutura de pastas

Dentro de cada volume, criei uma pasta por área:
- `E:\Financeiro`, `E:\RH-Sigiloso`
- `F:\Administracao`, `F:\Comercial`, `F:\TI`, `F:\Publica`

<p align="center"><img src="./imagens/11-criacao-pastas.png" width="1000"></p>

---

## Etapa 5 - Compartilhamento e permissões

Compartilhei cada pasta via SMB (perfil Quick), aplicando permissão de compartilhamento e NTFS pelos grupos de segurança do AD já existentes. As pastas `Financeiro` e `RH-Sigiloso` foram compartilhadas utilizando o caractere `$`, fazendo com que não apareçam na navegação pela rede.

O controle de acesso continua sendo realizado pelas permissões de compartilhamento e pelas permissões NTFS.

Grupos com Full Control em cada pasta:
- **Administracao**: Administração, Diretoria
- **Comercial**: Comercial, Diretoria
- **TI**: TI (Infraestrutura + Suporte Técnico), Diretoria
- **Publica**: Domain Users (Change), Diretoria (Full Control)
- **Financeiro$**: Diretoria
- **RH-Sigiloso$**: RH, Diretoria

A Diretoria tem acesso elevado a todas as pastas, refletindo a decisão tomada no laboratório anterior de não usar Domain Admins, e sim permissões pontuais pra demonstrar o princípio do menor privilégio.

<p align="center"><img src="./imagens/12-administracao.png" width="1000"></p>
<p align="center"><img src="./imagens/13-comercial.png" width="1000"></p>
<p align="center"><img src="./imagens/14-ti.png" width="1000"></p>
<p align="center"><img src="./imagens/15-publica.png" width="1000"></p>
<p align="center"><img src="./imagens/16-financeiro.png" width="1000"></p>
<p align="center"><img src="./imagens/17-rh-sigiloso.png" width="1000"></p>
<p align="center"><img src="./imagens/18-compartilhamentos-criados.png" width="1000"></p>

---

## Etapa 6 - Cotas com FSRM

Mesmo `Financeiro` e `RH-Sigiloso` estando no mesmo `Pool-Confidencial`, configurei cotas diferentes para cada pasta utilizando o FSRM.

- `Financeiro`: 10GB (Hard)
- `RH-Sigiloso`: 5GB (Hard)

<p align="center"><img src="./imagens/19-fsrm-cotas-confidencial.png" width="650"></p>

---

## Etapa 7 - Testes de acesso

Testei o acesso com dois usuários reais do domínio, logados no `WKS-TI-01`.

**Priscila Oliveira (grupo TI)**: acesso permitido à própria pasta (`TI`), negado nas demais (`Administracao`, `Financeiro$`, `RH-Sigiloso$`).

<p align="center"><img src="./imagens/20-teste-acesso-ti-permitido.png" width="850"></p>
<p align="center"><img src="./imagens/21-teste-acesso-administracao-negado.png" width="850"></p>
<p align="center"><img src="./imagens/22-teste-acesso-financeiro-negado.png" width="850"></p>
<p align="center"><img src="./imagens/23-teste-acesso-rh-sigiloso-negado.png" width="850"></p>

---

**Arthur Fernandes (Diretoria)**: acesso permitido em todas as pastas testadas, confirmando a permissão elevada configurada.

<p align="center"><img src="./imagens/24-teste-acesso-diretoria-administracao-permitido.png" width="850"></p>
<p align="center"><img src="./imagens/25-teste-acesso-diretoria-ti-permitido.png" width="850"></p>
<p align="center"><img src="./imagens/26-teste-acesso-diretoria-financeiro-permitido.png" width="850"></p>
<p align="center"><img src="./imagens/27-teste-acesso-diretoria-rh-sigiloso-permitido.png" width="850"></p>

Os testes com `Financeiro$` e `RH-Sigiloso$` foram realizados informando o caminho UNC diretamente, já que esses compartilhamentos não aparecem durante a navegação pela rede.

Mesmo conhecendo o caminho, apenas os grupos autorizados conseguiram acessar as pastas.

---

## Etapa 8 - Snapshot final

<p align="center"><img src="./imagens/28-snapshot.png" width="850"></p>

Snapshot `Storage-FileServer-Concluido`, criado após a validação dos compartilhamentos, permissões e testes de acesso, antes de iniciar o próximo laboratório de GPO.

---

## Conclusão

Esse laboratório saiu do plano original assim que tentei criar os Storage Pools: o Server Manager reconhecia só um dos quatro discos disponíveis, mesmo o PowerShell confirmando que todos estavam aptos. Investigando o problema, encontrei uma alternativa via PowerShell pra concluir a configuração dos pools.

Além disso, reutilizei toda a estrutura criada no laboratório anterior, aplicando os grupos de segurança do Active Directory diretamente nas permissões das pastas compartilhadas. Os testes realizados na estação `WKS-TI-01` confirmaram que cada usuário acessava apenas os recursos permitidos para seu grupo.

Com esse laboratório concluído, o ambiente ficou preparado para o próximo passo: mapear automaticamente esses compartilhamentos utilizando Group Policy.

## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)
