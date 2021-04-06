---
layout: post
title:  "Cometh : Les préparatifs" 
date:   2021-04-05 
categories: NFT cometh polygon
---

![]({{site.baseurl}}/assets/img/cometh_game.png){: width="96" }
Paré pour l'aventure de votre vie, vous vous dirigez déjà vers le lanceur qui vous transportera vers votre premier vaisseau de minage. Votre carrière de mineur de comètes va bientôt commencer ! Mais, avant de faire un pas de plus, êtes vous sûr d'être bien préparé ? C'est ce que nous allons voir dans cet article...

![Draft]({{site.baseurl}}/assets/img/logo_cometh.png){: width="132" }  
Avant de commencer, quelques précisions. Polygon (anciennement Matic) est une _solution de scalabilité_, en périphérie du réseau Ethereum principal (mainnet), moins cher en termes de frais et plus rapide. C'est pourquoi Cometh tourne sur Polygon. Dans les écosystèmes Polygon et Cometh, nous pourrons voir les termes Matic, Polygon et L2, qui sont équivalents dans ce contexte.  
Ce réseau étant assez récent, de nouveaux services viennent s'y installer au fil du temps, et ce billet n'a pas vocation à être exhaustif sur ces question.

## Prérequis
 - Polygon configuré dans Metamask
 - Des sous ($MATIC et $MUST)
 - Au moins un vaisseau

Nous allons voir dans le détail comment rassembler ces éléments.

## Configurer Metamask pour Polygon
<details markdown="1">
  {% include f/scal/polygon_confmetamask.md  %}
  <summary>
    Détails
  </summary>
</details>
Vérification du réseau : si le site du [jeu](https://game.cometh.io/) vous dit :  
![]({{site.baseurl}}/assets/img/cometh_wrongnetwork.png)  
C'est qu'il y a un problème ; vous n'êtes pas sur le bon réseau. Si par contre il affiche `No spaceships on Matic`, à ce stade c'est normal.

## Le Budget
Pour commencer, il faudra quelques centimes de $MATIC et quelques centimes de $MUST. Il faudra aussi **prévoir l'acquisition d'au moins un vaisseau, nous le verrons plus bas les différentes options**.
 - Le $MATIC est nécessaire pour interagir avec le réseau. L'ordre de prix d'une transaction est de 0.0001$MATIC, soit environ 0.000032EUR début avril 2021.
 - Le $MUST est nécessaire pour se déplacer. L'ordre de prix est fixé par les autres joueurs qui fixent leurs prix. Chaque déplacement coûtera typiquement entre 0.0003 et 0.01 $MUST, soit entre 0.06771 et 2.257 $EUR. Il pourrait vous falloir une dizaine de saut pour vous placer sur la bonne trajectoire.
En résumé, un minimum de 10$ sur le réseau Polygon devraient suffire pour jouer, hors frais d'acquisition de la flotte.

<details markdown="1">
  {% include f/scal/polygon_transfert.md  %}
  <summary>
    Transfert de valeurs sur Polygon
  </summary>
</details>
Pour des raisons d'économies de frais de réseau, il vaut mieux envoyer un seul token et convertir sur place selon nos besoins.  
Note : Le $MUST peut être transféré par le bridge Cometh (voir Transfert de vaisseaux plus bas)

## Acquisition de vaisseaux
Plusieurs possibilités s'offrent à nous.
### Sur Polygon
 - Acheter un vaisseau sur [opensea](https://matic.opensea.io/) (en version beta et en panne lors de la rédaction de ce billet)
 - [Louer un vaisseau](https://rental.cometh.io/) : la solution la moins onéreuse.
 - bientôt en _farmant_ du $DUST (méthode décrite dans la section mainnet plus bas)

### Sur le mainnet Ethereum
Les vaisseaux sont générés initialement sur le mainnet. Les déplacer vers Polygon vous en coûtera (d'après mon expérience) 0.004$Eth @100Gwei de gas.
Ceci précisé, voici les deux possibilités pour acheter un vaisseau :
 - Nous pouvons les obtenir en faisant du shopping sur [opensea](https://opensea.io/collection/cometh-spaceships) Attention : Vérifier dans la section `Chain Info` que le contrat est bien 0xbcd4F1EcFf4318e7A0c791C7728f3830Db506C71 , sinon c'est un faux !
 - Par le _[farming](https://www.cometh.io/farming/must)_ ; il faut déposer du $MUST, qui va générer quotidiennement du $DUST, non transmissible, qui permet d'acheter des vaisseaux. La fonction permettant d'accumuler du $DUST est la suivante : `DUST/min=(5*xMUST)/(20*xMUST+10000)`, selon la [documentation](https://medium.com/cometh/how-to-get-%EF%B8%8Fmust-and-earn-nft-spaceships-c2293c2a10b0).
Calculons la quantité de $DUST _farmée_ par jour avec 3 $MUST :
```
# MUST=3
# echo "(5*$MUST)/(20*$MUST+10000)*60*24" | bc -l
2.14711729622266400640
```
Note : La variété de vaisseaux disponibles en $DUST est plus restreinte que sur le marché.

### Transférer un vaisseau vers Polygon

<details markdown="1">
  {% include f/scal/cometh_transfert.md  %}
  <summary>
    Transfert de vaisseaux sur Polygon
  </summary>
</details>


## Services de change (Swap) et Farming
Nous trouverons plusieurs agents de change sur Polygon, et divers systèmes financiers viennent s'y installer. Les deux services de change historiques sont :
- [ComethSwap](https://swap.cometh.io/)
- [QuickSwap](https://quickswap.exchange/)

Il est intéressant de régulièrement déposer ses gains dans les fermes de ComethSwap.

## services Cometh 
[![Draft]({{site.baseurl}}/assets/img/cometh_apps.png){: width="600" }](https://app.cometh.io/)
