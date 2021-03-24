---
layout: post
title:  "Chats Lunaires : Bonus, Autres considérations"
date:   2021-03-16 20:15:39 +0200
categories: NFT mooncats etherscan smartcontracts
---

Autres considérations

# Renommer un chat

# Par contrat
Attardons-nous à présent aux lignes 3 et 14 de notre extrait du source. La 3 vérifie si le nombre total de chats à créer a été atteint (et refuse d'en créer un nouveau si c'est le cas), tandis que la 14 décrémente cette variable avant que la fonction n'envoye la nouvelle de l'heureux événnement au sauveteur du chat (ligne 16).

Et si...

Comme toujours, règle d'or, DYOR (Do Your Own Reasearch). Profitons-en le code est ouvert, et Code is Law.
interrogeons le contrat sur le nombre de Chats restants à produire !

Que dit [4. remainingCats](https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#readContract) ?
![]({{ site.baseurl }}/assets/img/mooncat_contract_remainingCats.png)
arf, ils avaient raison sur internet...


# ERC 20 
normes

### Read
https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#readContract

### Write
https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#writeContract

### Source
https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#code

### Un MoonCat dans un Wallet

## Le Wrapper
=> Partie II

## Analyse détaillée

## Exercices 
De quelle couleur est 0x00844fbf1c ?

## Aller plus loin
Analyse du smart contract pour les devs, en anglais sur [reddit]({{ site.baseurl }}/ressources/mooncats#developer-analysis-of-mooncat-contract) 
