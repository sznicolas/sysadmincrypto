---
layout: post
title:  "Chats Lunaires: Le Périple, deuxième partie"
date:   2021-03-16 20:15:39 +0200
permalink: TODO
categories: NFT mooncats etherscan smartcontracts
---


{% highlight solidity linenos %}
contract MoonCatsWrapped is ERC721 {

    using Counters for Counters.Counter;
    Counters.Counter private _tokenIdTracker;

    MoonCatsRescue public _moonCats = MoonCatsRescue(0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6);

    mapping(bytes5 => uint) public _catIDToTokenID;
    mapping(uint => bytes5) public  _tokenIDToCatID;
    string private _baseTokenURI;
    address public _owner = 0xD2927a91570146218eD700566DF516d67C5ECFAB;


    event Wrapped(bytes5 indexed catId, uint tokenID);
    event Unwrapped(bytes5 indexed catId, uint tokenID);

    constructor() ERC721("Wrapped MoonCatsRescue", "WMCR") {}

    function setBaseURI(string memory _newBaseURI) public {
        require(_msgSender() == _owner);
        _baseTokenURI = _newBaseURI;
    }

    function renounceOwnership() public {
        require(_msgSender() == _owner);
        _owner = address(0x0);
    }


    function _baseURI() public view virtual returns (string memory) {
        return _baseTokenURI;
    }


    function wrap(bytes5 catId) public {
        MoonCatsRescue.AdoptionOffer memory offer = _moonCats.adoptionOffers(catId);
        require(offer.seller == _msgSender()); //only owner can wrap
        _moonCats.acceptAdoptionOffer(catId);


        //check if it was previously assigned
        uint tokenID = _catIDToTokenID[catId];
        uint tokenIDToAssign = tokenID;

        if (tokenID == 0) {
            tokenIDToAssign = _tokenIdTracker.current();
            _tokenIdTracker.increment();
            _catIDToTokenID[catId] = tokenIDToAssign;
            _tokenIDToCatID[tokenIDToAssign] = catId;
        }
        _mint(_msgSender(), tokenIDToAssign);
        emit Wrapped(catId, tokenIDToAssign);

    }

    function unwrap(uint256 tokenID) public {
        bytes5 catId = _tokenIDToCatID[tokenID];
        address owner = ownerOf(tokenID);
        require(owner == _msgSender()); //only owner can unwrap
        _moonCats.giveCat(catId, owner);
        _burn(tokenID);
        emit Unwrapped(catId, tokenID);
    }

}
{% endhighlight %}

## interactions avec mooncat

## wrap

Attardons-nous à présent aux lignes 3 et 14 de notre extrait du source. La 3 vérifie si le nombre total de chats à créer a été atteint (et refuse d'en créer un nouveau si c'est le cas), tandis que la 14 décrémente cette variable avant que la fonction n'envoye la nouvelle de l'heureux événnement au sauveteur du chat (ligne 16).

Et si...

Comme toujours, règle d'or, DYOR (Do Your Own Reasearch). Profitons-en le code est ouvert, et Code is Law.
interrogeons le contrat sur le nombre de Chats restants à produire !

Que dit [4. remainingCats](https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#readContract) ?
![]({{ site.baseurl }}/assets/img/mooncat_contract_remainingCats.png)
arf, ils avaient raison sur internet...


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
