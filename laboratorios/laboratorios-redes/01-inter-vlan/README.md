<h1 align="center">Roteamento Inter-VLAN (Router-on-a-Stick)</h1>

<p align="center">
Laboratório prático de configuração de VLANs e roteamento Inter-VLAN utilizando <strong>Router-on-a-Stick</strong>, desenvolvido no Cisco Packet Tracer.
</p>


## Cenário

Imagine uma pequena empresa com dois departamentos:

- **VLAN 10** – TI
- **VLAN 20** – Financeiro

Cada departamento fica em uma rede diferente, então sem configuração nenhuma eles não conseguem se comunicar. Para resolver isso, configurei um roteador com subinterfaces para fazer o roteamento entre as duas VLANs — a técnica conhecida como Router-on-a-Stick.


## Tecnologias Utilizadas

- Cisco Packet Tracer
- Cisco IOS
- VLANs
- IEEE 802.1Q
- Router-on-a-Stick
- Inter-VLAN Routing


## Topologia

<p align="center">
<img src="imagens/01-topologia.png" width="1000">
</p>

---

## Endereçamento

### VLAN 10 - TI
| Dispositivo | Endereço IP |
|-------------|-------------|
| PC 1 | 192.168.10.11 |
| PC 2 | 192.168.10.12 |
| Notebook 1 | 192.168.10.21 |
| Notebook 2 | 192.168.10.22 |
| Gateway | 192.168.10.1 |

### VLAN 20 - Financeiro
| Dispositivo | Endereço IP |
|-------------|-------------|
| PC 3 | 192.168.20.11 |
| PC 4 | 192.168.20.12 |
| Notebook 3 | 192.168.20.21 |
| Notebook 4 | 192.168.20.22 |
| Gateway | 192.168.20.1 |


## Configuração do Switch

Criei as VLANs 10 e 20 para representar os departamentos da empresa e utilizei a VLAN 100 como VLAN nativa da conexão trunk entre o switch e o roteador.

- VLAN 10 – TI
- VLAN 20 – Financeiro
- VLAN 100 – Native VLAN

Na interface que liga o switch ao roteador, configurei modo trunk com encapsulamento IEEE 802.1Q e defini a VLAN 100 como nativa.

<p align="center">
<img src="imagens/02-config-switch.png" width="850">
</p>

---

## Configuração do Roteador

Configurei três subinterfaces no roteador utilizando encapsulamento IEEE 802.1Q, duas para as VLANs dos departamentos e uma para a VLAN nativa.

- G0/0/0.10
- G0/0/0.20
- G0/0/0.100 (Native VLAN)

Cada subinterface recebeu o IP de gateway correspondente à VLAN, permitindo que o roteador enxergasse as duas redes e fizesse o roteamento entre elas.

<p align="center">
<img src="imagens/03-config-roteador.png" width="850">
</p>

---

## Topologia Detalhada

Essa é a topologia final, já com as VLANs, a porta trunk e as subinterfaces do roteador configuradas.

<p align="center">
<img src="imagens/04-topologia-final.png" width="1000">
</p>

---

## Validação

Depois de finalizar a configuração, fiz alguns testes de ping entre as VLANs para verificar se o roteamento estava funcionando corretamente. Como os dispositivos conseguiram se comunicar, confirmei que o tráfego estava sendo encaminhado pelo roteador como esperado.

<p align="center">
<img src="imagens/05-teste-ping1.png" width="1200">
<br><br>
<img src="imagens/06-teste-ping2.png" width="1200">
</p>

---

## Competências Demonstradas

- Criação de VLANs
- Configuração de portas Access
- Configuração de portas Trunk
- VLAN Nativa
- IEEE 802.1Q
- Router-on-a-Stick
- Configuração de Subinterfaces
- Endereçamento IPv4
- Testes de conectividade

## Conclusão

Esse foi um dos primeiros laboratórios que montei no Cisco Packet Tracer e me ajudou a entender melhor como o Router-on-a-Stick funciona na prática. Ver a comunicação acontecendo entre redes diferentes deixou mais claro o papel das VLANs, da porta trunk e das subinterfaces do roteador.

Depois de configurar tudo e validar a comunicação entre os dispositivos, ficou mais fácil visualizar como esses componentes trabalham juntos para permitir o roteamento entre VLANs.

## Arquivos do laboratório

Caso queira reproduzir a configuração, o arquivo do Cisco Packet Tracer está disponível para download neste diretório.

- [lab-inter-vlan.pkt](./lab-inter-vlan.pkt)

 
## Autor

**Arthur Fernandes**

Estudante de Ciência da Computação, em transição de carreira para a área de TI (Suporte Técnico, Infraestrutura, Redes e NOC).

**LinkedIn:**
[Arthur Fernandes](https://www.linkedin.com/in/arthur-fernandes-289395272)