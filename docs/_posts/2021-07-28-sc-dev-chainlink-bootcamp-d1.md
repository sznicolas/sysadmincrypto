---
layout: post
title:  "Remix - Notes du Bootcamp Smart Contract Developer par Chainlink - Jour 1"
date:   2021-07-28
categories: dev remix solidity chainlink
permalink: notes_bc_chainlink_1
---

![]({{site.baseurl}}/assets/img/logo_chainlink.png){: width="48" }
![]({{site.baseurl}}/assets/img/logo_remix.png){: width="48" }
Déployer ses premiers Smart Contracts avec Remix.

Le bootcamp de Chainlink a été une expérience extrêmement enrichissante, très interactive, avec des personnes compétentes et motivées. Je remercie encore toute l'équipe et les participants, et je garde un excellent souvenir de ces journées !

Ce billet est inspiré des exercices mon bootcamp du 24.07.2021, avec pour objectif d'introduire Remix au lecteur. J'omets certaines parties du cours.

# Prérequis
Metamask configuré, aller sur le testnet Kovan. 
- Récupérer un peu d'$ETH sur le [faucet de Chainlink](https://linkfaucet.protofire.io/kovan).  
- Ouvrir [remix](https://remix.ethereum.org/).

# Premiers pas
## Rédaction du code source
Dans l'interface de Remix, créer un nouveau contrat : cliquer dans File Explorers..Contracts. Ajouter un fichier nommé MyFirstContract.  
![]({{site.baseurl}}/assets/img/remix-step1.png)  
Coller le source suivant :
{% highlight solidity linenos %}
pragma solidity ^0.6.7;

contract MyFirstContract {
    uint256 number = 0;
 
    function changeNumber(uint256 _num) public {
        number = _num;
    }
 
    function getNumber() public view returns (uint256){
        return number;
    }
}
{% endhighlight %}
Il ne nous reste plus qu'à compiler notre premier Smart Contract !
## Compilation
Dans la barre d'outils de gauche, cliquer le bouton suivant, `Solidity Compiler`.  
Sélectionner la version du compiler correspondant à ce que nous avons défini en la ligne 1 soit `0.6.7`, puis compiler.   
![]({{site.baseurl}}/assets/img/remix-step2.png)  
Ignorer l'éventuel warning concernant l'absence de licence.

## Exécution
Dans la barre d'outils cliquer le bouton suivant, `Deploy and Run transactions`.  
Pour cette première fois, nous allons l'exécuter localement.  
![]({{site.baseurl}}/assets/img/remix-step3.png)  
Jouer ensuite avec l'interface ;  
![]({{site.baseurl}}/assets/img/remix-step4.png)  

## Coder et tester
Revenir dans le `File Explorers` pour continuer l'implémentation ; ajouter des fonctions publiques, par exemple :

{% highlight solidity %}
    function reset() public {
        number = 0;
    }
{% endhighlight %}
Puis relancer la compilation, le déploiement et le run (voir section `Deployed Contracts`, chaque nouveau se trouve sous le précédant.  
    
{% highlight solidity %}
    function decreaseNumber() public {
        number -= 1;
    }

    function getNumberMultiplied(uint256 _multiplier) public view returns (uint256) {
        return number * _multiplier;
    }
{% endhighlight %}

## Déployer sur la blockchain
Bien vérifier dans Metamask que nous utilisons bien la bonne adresse et que nous sommes bien sur Kovan.  
![]({{site.baseurl}}/assets/img/remix-step5.png)  
Attendre la confirmation de Metamask...  
Copier l'adresse du contrat ![]({{site.baseurl}}/assets/img/remix-step6.png) et le chercher sur [etherscan](https://kovan.etherscan.io/).  
Félicitations, vous avez déployé votre premier Smart Contract !!!   
Jouer avec l'interface de Remix. À présent chaque transaction doit être validée par Metamask.

## Interagir avec un autre Smart Contract
Créer un nouveau Smart Contract en créant le nouveau fichier `Interaction.sol`.  
Pour la compilation, bien sélectionner le fichier que nous venons de créer, présenté dans la liste déroulante `CONTRACT` sous la forme `contrat (fichier.sol)`.  
![]({{site.baseurl}}/assets/img/remix-step7.png)  
{% highlight solidity linenos %}
pragma solidity ^0.6.7;

import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
 
contract PriceConsumerV3 {
 
    AggregatorV3Interface internal priceFeed;
 
    /**
     * Network: Kovan
     * Aggregator: ETH/USD
     * Address: 0x9326BFA02ADD2366b30bacB125260Af641031331
     */
    constructor() public {
        priceFeed = AggregatorV3Interface(0x9326BFA02ADD2366b30bacB125260Af641031331);
    }
 
    /**
     * Returns the latest price
     */
    function getLatestPrice() public view returns (int) {
        (
            uint80 roundID, 
            int price,
            uint startedAt,
            uint timeStamp,
            uint80 answeredInRound
        ) = priceFeed.latestRoundData();
        return price;
    }
}
{% endhighlight %}

Nécessite du $LINK (0.1 par appel, en obtenir sur le [faucet Chainlink](https://linkfaucet.protofire.io/kovan)) pour obtenir un retour.
{% highlight solidity linenos %}
import "@chainlink/contracts/src/v0.6/ChainlinkClient.sol";
 
contract APIConsumer is ChainlinkClient {
  
    uint256 public volume;
    
    address private oracle;
    bytes32 private jobId;
    uint256 private fee;
    
    /**
     * Network: Kovan
     * Chainlink - 0x2f90A6D021db21e1B2A077c5a37B3C7E75D15b7e
     * Chainlink - 29fa9aa13bf1468788b7cc4a500a45b8
     * Fee: 0.1 LINK
     */
    constructor() public {
        setPublicChainlinkToken();
        oracle = 0x2f90A6D021db21e1B2A077c5a37B3C7E75D15b7e;
        jobId = "29fa9aa13bf1468788b7cc4a500a45b8";
        fee = 0.1 * 10 ** 18; // 0.1 LINK
    }
    
    /**
     * Create a Chainlink request to retrieve API response, find the target
     * data, then multiply by 1000000000000000000 (to remove decimal places from data).
     ************************************************************************************
     *                                    STOP!                                         * 
     *         THIS FUNCTION WILL FAIL IF THIS CONTRACT DOES NOT OWN LINK               *
     *         ----------------------------------------------------------               *
     *         Learn how to obtain testnet LINK and fund this contract:                 *
     *         ------- https://docs.chain.link/docs/acquire-link --------               *
     *         ---- https://docs.chain.link/docs/fund-your-contract -----               *
     *                                                                                  *
     ************************************************************************************/
    function requestVolumeData() public returns (bytes32 requestId) 
    {
        Chainlink.Request memory request = buildChainlinkRequest(jobId, address(this), this.fulfill.selector);
        
        // Set the URL to perform the GET request on
        request.add("get", "https://min-api.cryptocompare.com/data/pricemultifull?fsyms=ETH&tsyms=USD");
        
        // Set the path to find the desired data in the API response, where the response format is:
        // {"RAW":
        //      {"ETH":
        //          {"USD":
        //              {
        //                  ...,
        //                  "VOLUME24HOUR": xxx.xxx,
        //                  ...
        //              }
        //          }
        //      }
        //  }
        request.add("path", "RAW.ETH.USD.VOLUME24HOUR");
        
        // Multiply the result by 1000000000000000000 to remove decimals
        int timesAmount = 10**18;
        request.addInt("times", timesAmount);
        
        // Sends the request
        return sendChainlinkRequestTo(oracle, request, fee);
    }
    
    /**
     * Receive the response in the form of uint256
     */ 
    function fulfill(bytes32 _requestId, uint256 _volume) public recordChainlinkFulfillment(_requestId)
    {
        volume = _volume;
    }
    
    /**
     * Withdraw LINK from this contract
     * 
     * NOTE: DO NOT USE THIS IN PRODUCTION AS IT CAN BE CALLED BY ANY ADDRESS.
     * THIS IS PURELY FOR EXAMPLE PURPOSES ONLY.
     */
    function withdrawLink() external {
        LinkTokenInterface linkToken = LinkTokenInterface(chainlinkTokenAddress());
        require(linkToken.transfer(msg.sender, linkToken.balanceOf(address(this))), "Unable to transfer");
    }
}
{% endhighlight %}

