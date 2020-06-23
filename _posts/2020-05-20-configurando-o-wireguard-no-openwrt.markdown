---
title: Configurando o Wireguard VPN no OpenWrt com Luci
date: "2020-05-21 09:23:26 -0300"
modified: 
layout: post
published: true
tags: OpenWRT, WireGuard, VPN
description: Veja como configurar o Wireguard VPN no OpenWrt.
---

Neste artigo vamos configurar um "servidor" VPN com WireGuard no OpenWrt e conectar um computador com [Arch Linux](https://www.archlinux.org){:target="blank"} (não demonstrarei instalação de pacotes) como exemplo de configurações a ser fazer para se ter a VPN ponta-a-ponta.

O [WireGuard](https://www.wireguard.com/){:target="blank"} é uma VPN extremamente simples, porém rápida e moderna, que utiliza criptografia de ponta-a-ponta para conectar os computadores através do túnel VPN. O WireGuard pretende ser tão fácil de configurar e implantar quanto o SSH. Uma conexão VPN é feita simplesmente trocando chaves públicas - exatamente como as chaves SSH - e todo o resto é tratado de forma transparente pelo WireGuard.

O [OpenWrt](https://www.openwrt.org/){:target="blank"} é um sistema operacional Linux direcionado a dispositivos incorporados, como roteadores. Ao usar o OpenWrt você tem total controle sobre o seu roteador, assim como num computador com [Linux](https://pt.wikipedia.org/wiki/Linux){:target="blank"}, e pode instalar pacotes para autmentar as funcionalidades do mesmo.

#### Instalando o WireGuard no OpenWrt

Para instalar o WireGuard no OpenWrt vamos utilizar o pacote `luci-i18n-wireguard-pt-br` que já instala todas a dependências necessárias, incluindo o próprio pacote `wireguard-tools` e o `luci-proto-wireguard` (necessário para a configuração gráfica na Luci).

<figure>
  <a href="https://i.imgur.com/GpO7Q4J.png">
    <img src="https://i.imgur.com/GpO7Q4J.png" alt="Pacotes instalados WireGuard">
  </a>
  <figcaption>Pacotes a serem instalados.</figcaption>
</figure>

Após feita a instalação reinicie o roteador para que as alterações de pacotes entrem em vigor pois quando se instala um pacote `lucil-proto-*` ele só irá aparecer se reinicar (se alguém souber de outra forma me avise que eu modifico).

#### Criando as chaves de acesso

Para conectar os pares a o WireGuard utiliza chaves de acesso semelhante ao SSH. Você deve criar uma chave privada (que deve ser mantida em segurança) e, através de sua chave privada, deve ser gerada a chave pública que você deve compartilhar com o ponto onde você vai se conectar.

Para criar as chaves usaremos o comando `wg` na em linha de comando (shell). Tanto faz você utilizar esses comandos na shell do OpenWrt ou do seu computador que já deve estar com o WireGuard instalado.

Criando as chaves para o servidor (adaptei os comandos da [página oficial](https://www.wireguard.com/quickstart/#key-generation){:target="blank"} para facilitar o entendimento):

```bash
$ wg genkey | tee serv-privatekey.txt | wg pubkey > serv-publickey.txt
```

Neste momento, onde você rodou esse comando, vão ter dois arquivos `.txt`, um com a chave privada e outro com a chave pública.

Repita o comando moficiado para gerar as chaves do cliente:

```bash
$ wg genkey | tee client-1-privatekey.txt | wg pubkey > client-1-publickey.txt
```

Para cada cliente a se conectar gere um par da chaves trocando o número `1` por outro ou (ou outra coisa) para você não sobrescrever as chaves anteriormente geradas.

#### Configurando a interface WireGuard no OpenWrt

Agora vamos configurar a interface para que o WireGuard seja executado. Vá em `Redes >> Interfaces` (presumindo que você está com sua interface Luci já em português) e clique em `Adicionar uma nova interface`, nomeie a interface,  selecione o protocolo `VPN WireGuard` e clique em `Criar interface`.

<figure>
  <a href="https://i.imgur.com/RpPiv26.png">
    <img src="https://i.imgur.com/RpPiv26.png" alt="Adicionar interfaces">
  </a>
  <figcaption>Adicionar interface.</figcaption>
</figure>

Na tela a seguir, coloque a chave privada so servidor (serv-privatekey.txt) no campo correspondente, escolha uma porta para o serviço rodar (ao que parece, na documentação, por padrão o WireGuard utilizaria a porta `51820`) e atribua um endereço de IP com `/24` para ser o endereço da interface a criada:

<figure>
  <a href="https://i.imgur.com/g3T6MCS.png">
    <img src="https://i.imgur.com/g3T6MCS.png" alt="Criando a interface">
  </a>
  <figcaption>Criando a interface.</figcaption>
</figure>

Vá na aba de `Configurações do Firewall` e atribua uma zona de Firewall para a interface. Eu normalmente gosto de criar uma zona própria para cada interface de rede. No restante desse artigo vou levar em conta que você também o fez:

<figure>
  <a href="https://i.imgur.com/g5eWkMg.png">
    <img src="https://i.imgur.com/g5eWkMg.png" alt="Configurações de firewall">
  </a>
  <figcaption>Zona de Firewall.</figcaption>
</figure>

Na aba `Pares` clique em `Adicionar parceiro`:

<figure>
  <a href="https://i.imgur.com/mLwSKjP.png">
    <img src="https://i.imgur.com/mLwSKjP.png" alt="Adicionar parceiro">
  </a>
  <figcaption>Adicionar parceiro.</figcaption>
</figure>

Nesta tela vamos adiconar as informações do dispositivo que irá se conectar a este OpenWrt. Coloque uma `Descrição`, insira a `Chave Pública` do cliente gerada (client-1-publickey.txt), atribua um `Endereço de IP Autorizado` e uma `Porta no ponto final` (endereço e porta que vamos configurar no cliente mais tarde). A opção `Manutenção da Conexão Persistente`, segundo a [documentação oficial](https://www.wireguard.com/quickstart/#nat-and-firewall-traversal-persistence){:target="blank"} serve para que, se você estiver se conectando através de NAT ou Firewall, a conexão não se "perca" e desconecte. Ser for o seu caso, coloque essa opção em `25`. Se o dispositivo parceiro a ser conectado tiver um endereço de IP válido você poderá colocá-lo em `Equipamento do ponto final` para aumentar a segurança (somente faça de for um IP válido e estático).

Adicone quantos parceiros você desejar. **Sempre que for para autorizar um novo dispositivo a se conectar a este OpenWrt é nesta tela que você fará isso.**

**Dica:** *Sempre que adicionar novos parceiros deve-se reinicar a interface do WireGuard clicando no botão `Reiniciar` na tela de interfaces.*

<figure>
  <a href="https://i.imgur.com/xNyfuM3.png">
    <img src="https://i.imgur.com/xNyfuM3.png" alt="Adiconar parceiro 2">
  </a>
  <figcaption>Adicionar parceiro 2</figcaption>
</figure>

Clique em `Salvar`.

Aplique as mudanças clicando em `Salvar & Aplicar`.

#### Configurações de Firewall

Feito isso já somos capazes de conectar na VPN pela rede local mas somente teremos acesso a própria rede da VPN e não acesso total as redes (rede lan, por exemplo).

Agora vamos liberar o acesso da rede VPN WireGuard para as outras redes. Vá em `Rede >> Firewall`. Vamos configurar as `Zonas de Firewall` para que permita `Encaminhar` pacotes da rede da `VPN` (`saron` nas imagens) para a zona da rede `lan` e vice versa, se assim você desejar. Habilite o encaminhamento para a zona da rede `wan` se você desejar que a VPN seja rota padrão do cliente e que ele "navegue" pela internet (utiliza-se muito o termo "sair pela rede") deste OpenWrt. Habilite o encaminhamento para todas as redes que você desejar. Ative também o `Mascaramento` na zona da interface da VPN.

<figure>
  <a href="https://i.imgur.com/W34QGo0.png">
    <img src="https://i.imgur.com/W34QGo0.png" alt="Zonas de Firewall">
  </a>
  <figcaption>Zonas de Firewall.</figcaption>
</figure>

<figure>
  <a href="https://i.imgur.com/AUXrvXb.png">
    <img src="https://i.imgur.com/AUXrvXb.png" alt="Zonas de firewall">
  </a>
  <figcaption>Zonas de Firewall.</figcaption>
</figure>

Clique em `Salvar`.

Aplique as mudanças clicando em `Salvar & Aplicar`.

Agora, quando nos conectarmos a VPN seremos capazes de "enxergar" as outras redes habilitadas no `Encaminhar` bem como sair pela `wan` do OpenWrt, se configurarmos como rota default nos clientes.

Na aba `Regras de Tráfego` devemos liberar a entrada de conexões para o dispositivo na porta configurada na interface `wan`.

<figure>
  <a href="https://i.imgur.com/MAdzQEd.png">
    <img src="https://i.imgur.com/MAdzQEd.png" alt="Firewall wan">
  </a>
  <figcaption>Firewall wan.</figcaption>
</figure>

Clique em `Adicionar` para adicionar uma nova regra de tráfego. Na tela que surgir dê um `Nome` para a regra, Selecione o `Protocolo` `UDP` (WireGuard roda no protocolo UDP!?), na `Zona de origem` escolha a zona da interface `wan`, na `Zona de destino` escolha `Dispositivo (entrada)`, em `Porta de destino` coloque a porta configurada no servidor OpenWrt (mesma tela onde utilizamos a chave privada do servidor).

<figure>
  <a href="https://i.imgur.com/7B8GIkN.png">
    <img src="https://i.imgur.com/7B8GIkN.png" alt="Entrada de Firewall">
  </a>
  <figcaption>Entrada de Firewall.</figcaption>
</figure>

Clique em `Salvar`.

Aplique as mudanças clicando em `Salvar & Aplicar`.

Em `Condição Geral >> Condição geral do WireGuard` você deverá ver uma tela como a abaixo onde aparece a chave pública do seu servidor e dos parceiros conectados. Nesta tela você sempre pode verificar quais os parceiros estão conectados no momento.

<figure>
  <a href="https://i.imgur.com/0ckyU6g.png">
    <img src="https://i.imgur.com/0ckyU6g.png" alt="Condição geral do WireGuard">
  </a>
  <figcaption>Condição geral do WireGuard.</figcaption>
</figure>

#### Configurando a conexão em um parceiro

Aqui utilizarei um computador com Arch Linux com a interface [GNOME](https://gnome.org/){:target="blank"}. Você pode aplicar os mesmos princípos para conectar vários tipos de dispositivos, inclusive outro OpenWrt (imagine que show fica!).

No [site oficial](https://www.wireguard.com/install/){:target="blank"} do WireGuard tem as instruções e links para download e instalação dos aplicativos para vários sistemas operacionais.

Ao criar uma nova interface no [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager_(Portugu%C3%AAs)){:target="blank"} do GNOME, ao criar uma nova conexão VPN eu tenho a tela de configurações da interface WireGuard. O conceitos dessas informações podem ser aplicados a todos os clientes de todas as plataformas.

Darei um `Nome` a interface, em `Addresses` colocarei o endereço de IP autorizado e em `Listen Port` a porta (ambos configurados na configuração do parceiro no OpenWrt), em `Private Key` colocaremos a chave privada do parceiro (client-1-privatekey.txt), em `DNS servers` utilizaremos o endereço de IP configurado na interface do OpenWrt (192.168.3.1) para que o dispositivo possa ser capaz de usar a rede do OpenWrt como rota padrão e "saia" por ela para a internet.

<figure>
  <a href="https://i.imgur.com/etsL8Wh.png">
    <img src="https://i.imgur.com/etsL8Wh.png" alt="NetworkManager">
  </a>
  <figcaption>Tela do NetworkManager.</figcaption>
</figure>

Se no dispositivo que você está configurando tiver a opção de adicionar um parceiro, clique e vamos adicionar o OpenWrt como parceiro deste dispositivo. Em `Publick Key` colocaremos a chave pública do OpenWrt (serv-publickey.txt), em Allowed IPs colocaremos `0.0.0.0/0`, em `Endpoint` colocaremos o **endereço de IP válido** do servidor (interface `wan` do OpenWrt) e a porta que configuramos e liberamos no Firewall na sintaxe `IP:PORTA`.

<figure>
  <a href="https://i.imgur.com/ntJ3Hhb.png">
    <img src="https://i.imgur.com/ntJ3Hhb.png" alt="Tela de Parceiros">
  </a>
  <figcaption>Tela de parceiros.</figcaption>
</figure>

Pronto. Já devemos ser capazes de conectar e navegar pela VPN. Você pode conferir a conexão na tela de `Condição geral do WireGuard`.

<figure>
  <a href="https://i.imgur.com/i6IliFr.png">
    <img src="https://i.imgur.com/i6IliFr.png" alt="Condição geral do WireGuard">
  </a>
  <figcaption>Condição geral do WireGuard.</figcaption>
</figure>

Veja que agora o Parceiro `celular-mariel` está conectado e as informações da conexão são mostradas.

O dispositivo conectado (celular-mariel) está com a rota padrão saindo pelo OpenWrt. Você pode tentar navegar na internet agora e verificar o IP que você está "saindo" através de sites como o [meu ip](https://www.meuip.com.br/){:target="blank"}. O IP mostrado aqui deverá ser o IP válido do OpenWrt.

#### Bônus: Alterando rota padrão no clientes

Se você não deseja que a VPN seja utilizada como rota padrão no cliente você deverá utilizar esses passos.

Para quê serve isso? Uma rota padrão é por onde você "navega na internet". Se deixarmos a VPN como rota padrão TODAS as conexões com a internet serão feitas por dentro da VPN.

Ao alterar isso faremos com que somente as conexões a endereços de IP da nossa rede VPN ou das nossas redes internas do OpenWrt trafeguem por dentro da VPN e outras conexões (navegação normal na internet) ocorram pela conexão onde o dispositivo está conectado.

Você deve alterar a opção Allowed IPs que está em `0.0.0.0/0` e colocar as redes utilizadas no OpenWrt, separadas por vírgula, no meu caso `192.168.3.0/24, 192.168.1.0/24`.

Você também deverá configurar manualmente rota para as redes que não sejam a da conexão VPN (no meu caso ai, `192.168.1.0/24`). No `NetworkManager` isso se faz na mesma janela de configuração da VPN na aba `IPv4`.

<figure>
  <a href="https://i.imgur.com/Y0YnBHu.png">
    <img src="https://i.imgur.com/Y0YnBHu.png" alt="NetworkManager">
  </a>
  <figcaption>Tela do NetworkManager.</figcaption>
</figure>

No aplicativo cliente para celulares [Android](https://play.google.com/store/apps/details?id=com.wireguard.android){:target="blank"}, na tela de edição da conexão, clique em `ALL APPLICATIONS`, na tela que surgir clique na aba `INCLUDE ONLY` e selecione somente os aplicativos que deverão sair pela VPN.

<figure>
  <a href="https://i.imgur.com/ZVA6uOr.png">
    <img src="https://i.imgur.com/ZVA6uOr.png" alt="Aplicativo celulare">
  </a>
  <figcaption>Aplicativo celular.</figcaption>
</figure>

<figure>
  <a href="https://i.imgur.com/4YcmrjY.png">
    <img src="https://i.imgur.com/4YcmrjY.png" alt="Aplicativo celular">
  </a>
  <figcaption>Aplicativo celular.</figcaption>
</figure>

**Consulte a documentação de seu dispositivo/sistema sobre como fazer isso.**
