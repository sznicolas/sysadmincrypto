---
layout: fiche
title:  "Utilisation pratique de zkSync"
date:   2021-03-22 13:15:39 +0200
categories: fiches zksync
permalink: fzksync
---

![logo gitcoin]({{site.baseurl}}/assets/img/logo_zkSync.png){: width="96" }

_En cours de rédaction..._

## Présentation
_En cours de rédaction..._  
C'est un layer 2, une couche périphérique à la blockchain Ethereum, avec laquelle elle se synchronise.
Solution de scalabilité permettant de réduire notablement les frais de réseau. Ces frais sont payés dans la devise du token.

## Transférer des tokens
1. Se rendre sur [l'interface](https://zksync.io/)
2. Open wallet
3. Connecter son wallet
4. Signer la transaction
5. Nous pouvons voir notre wallet. Ici, celui de SysAdmInCrypto, déséspérément vide :-(  
![]({{site.baseurl}}/assets/img/zksync_empty_wallet.png)
6. Ajoutons des fonds. Ici, des DAI.
7. Unlock ; autoriser le contrat à interagir avec ce token dans ce wallet `Frais 0.00532296 Ether ($9.07) à 120 Gwei`
Puisque les frais seront payés avec ce token, il est inutile de transférer de l'Eth.
8. Déposer le token `Frais 0.0083115 Ether ($14.21) à 100 Gwei`
![]({{site.baseurl}}/assets/img/zksync_deposit.png)
9. Une fois la transaction validée, les fonds sont visibles sur le L2.  
![]({{site.baseurl}}/assets/img/zksync_wallet.png)
