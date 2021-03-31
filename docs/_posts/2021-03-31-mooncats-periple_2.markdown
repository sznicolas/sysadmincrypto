---
layout: post
title:  "Chats Lunaires: Le Périple, deuxième partie"
date:   2021-03-31 08:00:00 +0200
permalink: mooncatpartii
categories: NFT mooncats etherscan smartcontracts
---

![un Chat Lunaire]({{site.baseurl}}/assets/img/mooncat_rarible.png){: width="96" }
Lors de notre [précédante exploration]({{ site.baseurl }}/mooncatparti), nous avions vu comment était généré un Chat Lunaire. A présent voyons comment fonctionne un _wrapper_, qui annonce emballer notre chat, le normaliser en NFT ERC721... Alors que la réalité est toute autre ! Il veut le cryogéniser et le cloner ! Voyez plus bas, la suite va vous étonner !

## Le NFT ERC721
Avant d'entrer un peu plus dans les détails du wrapper, attardons-nous sur ce qu'est un ERC721, puisqu'il va créer un token de type ERC721 à partir d'un MoonCat.  
Traduisons l'introduction de cette [norme](https://eips.ethereum.org/EIPS/eip-721) : "_Le standard suivant permet l'implémentation d'une API standard au sein des smart contracts. Ce standard fournit des fonctionnalités de base pour traquer et transférer des NFT._"  
Cette norme permet donc de définir des des NFT standardisés, simplifiant les échanges sur les places de marché telles que [Opensea](https://opensea.io/assets/wrapped-mooncatsrescue) et [Rarible](https://rarible.com/collection/0x7c40c393dc0f283f318791d746d894ddd3693572),  à travers la définition de [méthodes et attributs](https://fr.wikipedia.org/wiki/Programmation_orient%C3%A9e_objet#Objet_(attributs_et_m%C3%A9thodes)) génériques. Si l'objet est conforme, il est également visible dans le wallet depuis etherscan :  
![]({{site.baseurl}}/assets/img/mooncat_etherscan_tab_erc721.png)  
Mais revenons à notre _wrapper_, dont la tâche sera de "dresser" les Chats Lunaires selon la norme ERC721, tout en en conservant leurs attributs. Je mets le verbe "dresser" entre guillemets, car tout amateur de Chat qui se respecte est bien conscient qu'un Chat ne se laissera jamais "dresser". Surtout s'il est issu d'un smart contract sauvage...

## Le wrapper
 
Avant de commencer, rappelons-nous la procédure de JR_Kunz pour [wrapper les MoonCats](https://twitter.com/RJ_Kunz/status/1370508875598807041) évoqué dans notre [premier billet]({{ site.baseurl }}/mooncatintro). Traduction libre :
1. Activer dans le contrat sur la page etherscan de [MoonCatsRescue](https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#writeContract) 12. makeAdoptionOffer, avec comme paramètre le `catId` et l'adresse du contrat du _wrapper_,
2. Activer la fonction `wrap()` du _[wrapper](https://etherscan.io/address/0x7c40c393dc0f283f318791d746d894ddd3693572#writeContract)_ le `catId`

Étudions à présent un extrait de code de [Smart Contract MoonCatsWrapped](https://etherscan.io/address/0x7c40c393dc0f283f318791d746d894ddd3693572#code) à partir de la ligne 1800, soit la ligne 1 de notre extrait.
{% highlight solidity linenos %}
contract MoonCatsWrapped is ERC721 {

    using Counters for Counters.Counter;
    Counters.Counter private _tokenIdTracker;

    MoonCatsRescue public _moonCats = MoonCatsRescue(0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6);

    mapping(bytes5 => uint) public _catIDToTokenID; // dictionnaire (hash) convertissant catID vers ID ERC721
    mapping(uint => bytes5) public  _tokenIDToCatID; // dictionnaire de ID ERC721 vers catID
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
        require(offer.seller == _msgSender()); //seul le propriétaire peut emballer son Chat
        _moonCats.acceptAdoptionOffer(catId);


        //Vérifie si le chat n'est pas déjà emballé
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
        require(owner == _msgSender()); //seul le propriétaire peut déballer son chat
        _moonCats.giveCat(catId, owner);
        _burn(tokenID); // on brûle le chat ERC721 🔥😿🔥
        emit Unwrapped(catId, tokenID); // on dégèle le Chat initial ❄️
    }

}
{% endhighlight %}

### wrap()
* Ligne 1 : la classe hérite de ERC721, classe de base décrite ligne 1321 et suivantes, héritant elle-même de plusieurs classes.
* Ligne 6 : récupération du contrat `MoonCatsRescue` dans `_moonCats`, le Contrat Fondateur des Chats Lunaires, afin de pouvoir agir avec lui.
* Ligne 35 : `function wrap(bytes5 catId) public` : c'est dans cette fonction que l'essentiel se passe !
  * Ligne 36 : récupère (l'innocente) offre d'adoption de l'autre contrat.
  * Ligne 38 : Le Chat est adopté par le contrat _wrapper_ et ne pourra en sortir uniquement lorsque la fonction `unwrap()` sera appelée. Le Chat est donc _Freezed_, ce qui justifie le chapeau accrocheur de cet article :-)
  * Lignes 46 à 48 : assignation d'un numéro unique au Chat à emballer (c'est un compteur incrémenté ligne 47). C'est donc ici qu'est construite une table de correspondance entre un `catID` et le tokenID qui lui est associé lors du _wrap_ en ERC721.
  * Ligne 49 : on récupère l'ADN du chat (son `catId`).
Ouvrons une parenthèse sur la question de l'ADN du Chat. La place nous manque pour tout analyser dans le détail, basons-nous sur ces [deux](https://www.reddit.com/r/MoonCatRescue/comments/6tkdl0/developer_analysis_of_mooncat_contract/) [sources](https://www.reddit.com/r/MoonCatRescue/comments/m4gazs/mooncat_k_values_explained/) ;  le catID d'un Chat "normal" se décompose de la manière suivante : `0x00ddrrvvbb` où :
    * `0x00` est le préfixe (héxadécimal, 00). Il en existe des différents, les genesis, sortes d'albinos qui devaient revenir aux créateurs mais dont beaucoup ont été perdus... Mais c'est une autre histoire.
    * `dd` l'aspect général du Chat : 
![]({{ site.baseurl }}/assets/img/mooncat_kval.jpg)  
([source](https://www.reddit.com/r/MoonCatRescue/comments/m4gazs/mooncat_k_values_explained/))  
    * `rrvvbb` la couleur de sa robe en rvb.
  * Lignes 51 et 52, le token ERC721 est forgé, et l'information est envoyée sur la blockchain.

Exemple de [transaction](https://etherscan.io/tx/0xc1777250b8731637bccf8a47503006aab29049a53f6416580fb172ed367e4b01). L'onglet ([Logs](https://etherscan.io/tx/0xc1777250b8731637bccf8a47503006aab29049a53f6416580fb172ed367e4b01#eventlog)) permet de voir les paramètres utilisés, en particulier l'initiateur de la transaction, le `catID` et le `tokenID` correspondant qui lui assigné.

Dans le wallet du contrat de [wrap](https://etherscan.io/tokenholdings?a=0x7C40c393DC0f283F318791d746d894DdD3693572), nous pouvons observer les Chats cryogénisés (càd gelés dans le compte du contrat jusqu'à l'appel d'un `unwrap()`)
![]({{ site.baseurl }}/assets/img/mooncat_etherscan_view_erc20.png)  

### unwrap()
* Ligne 56 : `function unwrap(uint256 tokenID) public` : opération inverse, on décongèle le Chat.
* Ligne 60 : Interaction avec le smart contract `MoonCatsRescue` pour remettre le Chat original à son propriétaire.
* Ligne 61 : `_burn(tokenID)` : le token représentant le Wrapped MoonCatsRescue. La chaleur dégagée dégèle le Chat précédemment cryogénisé, le Chat original est prêt à être émis.  
* Ligne 62 : Émission sur la blockchain de l'ensemble la transaction d'unwrap.  

Exemple de [transaction](https://etherscan.io/tx/0x2c6fc69c46d4afe651ee86b0360704b6422a5602ef1d891eefaf0bc417a66199). Pour retrouver l'appel à `unwrap` et connaître les paramètres TokenID et catID, voir en bas de page de l'onglet Logs(5).

## Remonter l'historique d'un Chat
A partir d'un wallet possédant un Chat (par exemple sur [Rarible](https://rarible.com/collection/0x7c40c393dc0f283f318791d746d894ddd3693572)) ou [Opensea](https://opensea.io/assets/wrapped-mooncatsrescue), nous pouvons remonter l'historique, à l'aide d'etherscan. Nous pouvons également récupérer son TokenID et interroger le [contrat](https://etherscan.io/address/0x7c40c393dc0f283f318791d746d894ddd3693572#readContract) ; fonctions 5 et 11 par exemple.  
![]({{ site.baseurl }}/assets/img/mooncat_rarible.png)  
Ici le `7519`. Si à partir du TokenID nous pouvons obtenir le catId, nous pouvons aussi aller glaner des informations sur le contrat [mooncatrescue](https://etherscan.io/address/0x60cd862c9C687A9dE49aecdC3A99b74A4fc54aB6#readContract) (attention à la présentation des valeurs ; le TokenID est un uint256, donc un entier non signé, alors que le catId, un bytes5, doit être préfixé de `0x`)


## Aller plus loin (en anglais)
* Un excellent article sur le frontrunning exploitant une faille dans le process d'adoption de Chat sur [reddit](https://www.reddit.com/r/MoonCatRescue/comments/m5qjeg/frontrunning_a_primer/).
* Analyse du smart contract pour les devs sur [reddit]({{ site.baseurl }}/ressources/mooncats#developer-analysis-of-mooncat-contract) 
* Statistiques de types de Chats ayant été générés [reddit](https://www.reddit.com/r/MoonCatRescue/comments/m3t7nv/mooncat_numbersrarities/)
* [The Non-Fungible Token Bible](https://opensea.io/blog/guides/non-fungible-tokens/)
* Un thread très complet à èrpèps des NFT sur [twitter](https://twitter.com/minionabct/status/1364199356245495808)

