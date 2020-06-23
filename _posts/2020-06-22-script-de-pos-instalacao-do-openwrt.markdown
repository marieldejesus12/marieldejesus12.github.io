---
title: "Script de pós instalação do OpenWRT"
date: "2020-06-22 19:09:50 -0300"
modified: 
layout: post
published: true
tags: OpenWRT, Script, Ferramentas
description: Tutorial de utilização do openWRTAfterInstall.sh, um script de pós instalação do OpenWRT.
---

Um dia desses, em conversas no grupo do telegram do [OpenWRT](https://t.me/Openwrt_Lede_Librecmc_Brasil){:target="blank"}, falávamos, entre outras coisas, sobre dificuldades de ter que refazer a instalação do OpenWRT por completo, por qualquer motivo que seja. Configuração de [EXTROOT](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration){:target="blank"}, configurações de redes, instalação de pacotes, configurações de rotas e por ai vai. Quando se precisa refazer uma instalação assim é normal que dê trabalho. **Mas e se esse trabalho pudesse ser de alguma forma amenizado pela criação de scripts?**

Pensando nisso desenvolvi o [openWRTAfterInstall.sh](https://gitlab.com/marieldejesus12/openwrtafterinstall){:target="blank"}. Um script que ajudará nas configurações do OpenWRT.

O script faz muitas configurações automáticas no OpenWRT. Atualmente o script [openWRTAfterInstall.sh](https://gitlab.com/marieldejesus12/openwrtafterinstall){:target="blank"} é capaz de:

- Configurar [EXTROOT](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration){:target="blank"}
- Configurar [SWAP](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration#devices_32_mb_ram){:target="blank"} no [EXTROOT](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration){:target="blank"}
- Instalar pacotes de tradução pt-br da [Luci](https://openwrt.org/docs/guide-user/luci/start){:target="blank"}
- Instalar [Adblock](https://github.com/openwrt/packages/tree/master/net/adblock/files){:target="blank"}
- Instalar [banIP](https://github.com/openwrt/packages/tree/master/net/banip/files){:target="blank"}
- Instalar [HD-Idle](https://openwrt.org/docs/guide-user/storage/hd-idle){:target="blank"}
- Instalar [SQM Scripts](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm){:target="blank"}
- Instalar [Transmission](https://transmissionbt.com/){:target="blank"}
- Instalar os scripts do [YouTube Pihole Adblock](https://gitlab.com/marieldejesus12/youtube-pihole-adblock){:target="blank"}

Se você tiver alguma sugestão de inclusão de funcionalidades pode entrar em contato comigo pelo [Telegram](https://t.me/marieldejesus12){:target="blank"} ou pode abrir uma [issue](https://gitlab.com/marieldejesus12/openwrtafterinstall/-/issues/new){:target="blank"} na página do projeto.

**Para usar o script é muto simples: baixe o script e execute através do comando abaixo.**

```bash
wget https://gitlab.com/marieldejesus12/openwrtafterinstall/-/raw/master/openWRTAfterInstall.sh -O /tmp/openWRTAfterInstall.sh && sh /tmp/openWRTAfterInstall.sh
```

Essas são algumas telas que temos hoje no script:

<figure>
  <a href="https://i.imgur.com/hXsUbpN.png">
    <img src="https://i.imgur.com/hXsUbpN.png" alt="Configuração do EXTROOT">
  </a>
  <figcaption>Configuração do EXTROOT</figcaption>
</figure>
<figure>
  <a href="https://i.imgur.com/aLN110N.png">
    <img src="https://i.imgur.com/aLN110N.png" alt="Configuração da SWAP no EXTROOT">
  </a>
  <figcaption>Configuração da SWAP no EXTROOT</figcaption>
</figure>
<figure>
  <a href="https://i.imgur.com/B9FnR1D.png">
    <img src="https://i.imgur.com/B9FnR1D.png" alt="Tela do Menu Principal">
  </a>
  <figcaption>Tela do Menu Principal</figcaption>
</figure>
<figure>
  <a href="https://i.imgur.com/pmQE3x6.png">
    <img src="https://i.imgur.com/pmQE3x6.png" alt="Menu de instalação de Programas">
  </a>
  <figcaption>Menu de instalação de Programas</figcaption>
</figure>
<figure>
  <a href="https://i.imgur.com/O8eejC9.png">
    <img src="https://i.imgur.com/O8eejC9.png" alt="Menu de instalação do script YouTube Pihole Adblock">
  </a>
  <figcaption>Menu de instalação do script YouTube Pihole Adblock</figcaption>
</figure>
