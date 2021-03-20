---
layout: post
title:  "Chats Lunaires: Le Périple, première partie"
date:   2021-03-20 20:15:39 +0200
permalink: mooncatparti
categories: NFT mooncats etherscan smartcontracts
---

![Lune]({{site.baseurl}}/assets/img/mooncat_searching.png){: width="96" }
Suivons maintenant les aventures d'un Chat Lunaire, depuis la petite graine fabriquée sur la lune à jusqu'à son douillet wallet, en passant par sa gestation, ses premiers pas de simple token puis son enrobage ERC721 ;-)

_Cet article fait suite à [Le Grand Sauvetage des Chats Lunaires]({{ site.baseurl }}/mooncatintro)_
## Génération de la petite graine

La graine était générée sur le [fontend](https://mooncatrescue.com/scan.html), un bytes32. Une seed ressemble à ceci : `0x9750622ea94fe1312c8b958bb7d442c010e13d8921dcb9428789d7377a810f08` (soit 32 octets en [hexadécimal](https://fr.wikipedia.org/wiki/Syst%C3%A8me_hexad%C3%A9cimal). Ici, le premier octet est 97, soit 151 en décimal).
L'interface a malheureusement été modifiée (tous les chats ayant été sauvés) et n'offre plus que l'identifiant (catId).  
![]({{ site.baseurl }}/assets/img/mooncat_found.png)  
Pour exemple, un catId : `id: 0x002d8d227d`.

## Visite de la forge MoonCats
_Petite analyse partielle du smart contract Mooncats ([0x60cd...54aB6](https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6))_

_Se reporter à la fiche décrivant l'utilisation d'[etherscan]({{ site.baseurl }}/fetherscan#rechercher-une-adresse) pour plus de détails._

Par contrat :-), la forge a définitivement stoppé son activité de production. On peut encore la visiter, mais seules les activités administratives sont encore en fonction. Avant de visiter l'ancienne forge, arrêtons-nous questionner le service commercial ;-) Cherchons de l'information sur notre catId `0x002d8d227d`.

### Interroger un contrat
Demande d'information... Comment s'appelle-t-il ? 
![]({{ site.baseurl }}/assets/img/mooncat_etherscan_querydetails.png)  
_[readContrat](https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#readContract) 6. getCatDetails_

**name** _bytes32_ : `0x6e79616e00000000000000000000000000000000000000000000000000000000`

Mais encore ? Décodons-le...
```
$ echo 0x6e79616e00000000000000000000000000000000000000000000000000000000 | xxd -rp
nyan
$ 
```
![]({{ site.baseurl }}/assets/img/mooncat_decodename.png)  
Note : Lire un contrat ne nécéssite pas de connection à son wallet et n'engendre pas de frais de gas ; nous ne faisons que lire des informations sur la blockchain.

Mais j'entends d'ici votre impatience, allons voir la forge !

### La forge
Regardons dans le smart contract la partie qui nous intéresse, la fonction `rescueCat()`, ligne 71 et suivantes, reproduites ci-dessous (les numéros de lignes de la suite de l'article feront référence à cet extrait du [code source](https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#code)):

{% highlight solidity linenos %}
  /* registers and validates cats that are found */
  function rescueCat(bytes32 seed) activeMode returns (bytes5) {
    require(remainingCats > 0); // cannot register any cats once supply limit is reached
    bytes32 catIdHash = keccak256(seed, searchSeed); // generate the prospective catIdHash
    require(catIdHash[0] | catIdHash[1] | catIdHash[2] == 0x0); // ensures the validity of the catIdHash
    bytes5 catId = bytes5((catIdHash & 0xffffffff) << 216); // one byte to indicate genesis, and the last 4 bytes of the catIdHash
    require(catOwners[catId] == 0x0); // if the cat is already registered, throw an error. All cats are unique.

    rescueOrder[rescueIndex] = catId;
    rescueIndex++;

    catOwners[catId] = msg.sender;
    balanceOf[msg.sender]++;
    remainingCats--;

    CatRescued(msg.sender, catId);

    return catId;
  }
{% endhighlight %}

Ligne 2 : la définition de fonction. La fonction accepte un paramètre, la `seed` de type `bytes32`, générée par le frontend.  
Ligne 18 : nous lisons qu'elle renvoie le fameux catId, qui est généré lignes 4 à 6.  
L'heureux événement se trouve ligne 16 ; l'annonce officielle au monde de la naissance du chat à telle adresse. Dans le contexte de MoonCatRescue : **Un Chat a été Sauvé !!!**  😹  
Notons ici deux points sur cette ligne 16 `CatRescued(msg.sender, catId);` :
1. elle appelle `event CatRescued(address indexed to, bytes5 indexed catId);` en ligne 54 du contrat original (non reproduite ici)
2. la notation est désormais obsolète et devrait être précédée du mot-clé `emit` donc nous devrions écrire `emit CatRescued(msg.sender, catId);`
C'est cet `event` qui publiera sur la blockchain.

Pour résumer, il fut un temps où vous pouviez sauver un chat abandonné sur la lune en saisissant une graine dans le smart contract.  
1. Activer rescueCat ![]({{ site.baseurl }}/assets/img/mooncat_etherscan_write_rescueCat.png)  
2. La transaction qui en découle ![]({{ site.baseurl }}/assets/img/mooncat_etherscan_transaction.png)  
[transaction complète](https://etherscan.io/tx/0x889fbba3941096949840bce5292a0e6fad39f1da757f3f3563066754d23c2b3f)  

Note : voir fiche pratique [interagir avec un smartcontract dans etherscan]({{ site.baseurl }}/fetherscan#interagir-avec-un-contrat)

## A Suivre...
Dans le prochain épisode, nous nous attarderons sur la suite du périple. Emballage de chat et shopping seront au programme !

## Bonus
Comment voir à quoi ressemble le chat de notre dernier exemple ? Son `catId` émis par `CatRescued` doit se trouver sur la blockchain. Fouillons un peu... 
![]({{ site.baseurl }}/assets/img/mooncat_etherscan_transaction_logs.png)

Gardons la valeur `006060DFEC` et collons-le à l'URL https://mooncats.netlify.app/#/cat/ , préfixé de `0x` (signifiant : "ce qui suit est de l'héxa") 
![]({{ site.baseurl }}/assets/img/mooncat_viewer.png)  
[Voir la page complète](https://mooncats.netlify.app/#/cat/0x006060DFEC)
