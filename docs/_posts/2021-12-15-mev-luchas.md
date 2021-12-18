---
layout: post
title:  "Il faut sauver le Luchador #5694 (mev, bundle de transactions)"
date:   2021-12-17 01:00:00 +0200
categories: ethereum mev nft
---

![]({{site.baseurl}}/assets/img/lucha_prison.png){: width="96" }
Comment exfiltrer un NFT d'une adresse compromise ?

# Contexte
Un membre de la communauté des [Luchadores](https://luchadores.io) a eu la mauvaise idée de donner sa _seed phrase_ à un inconnu... _Rekt_.(Son histoire [ici](https://www.youtube.com/watch?v=WGNTwnSWYQc)).  
Ses tokens ERC20 et son Ether ont été siphonnés. Seul reste un NFT, son Luchador. Dans un geste désespéré il envoie de quoi payer le gas pour transférer son Lucha en lieu sûr.  
5 blocks après réception des fonds, soit 67 secondes, _Rekt again_.  

![]({{site.baseurl}}/assets/img/lucha_rekt.png)  
Une transaction transfère le tout, au Wei près, vers une autre adresse. Nous pouvons raisonnablement penser qu'un automate _veille_ sur son adresse, prêt à agir.  

Comment récupérer le Luchador ? Il faut pouvoir payer les gas pour effectuer la transaction de sauvetage sans qu'il puisse y avoir de transfert sortant depuis l'adresse compromise.  
En d'autres termes, nous voudrions que :
1. Alice sponsorise Charlie en lui envoyant des fonds pour payer le gas.
2. Charlie envoye son NFT en lieu sûr à Bob.

Nous craignons que :
1. Alice sponsorise Charlie en lui envoyant des fonds pour payer le gas.
2. Robert, qui a la clé privée de Charlie, se précipite et le pille (tel l'URSSAF sur un indépendant).
3. Charlie est toujours à sec `¯\_(ツ)_/¯`

Le cœur du problème est que nous n'avons pas le pouvoir de coordonner deux transactions depuis deux adresses différentes. Ce pouvoir est le privilège des mineurs.  
Lorsqu'on signe une transaction, elle est envoyée dans le [_mempool_](https://cryptoast.fr/mempool-blockchain/) et les mineurs y choisissent celles qu'ils vont intégrer dans le prochain bloc. Robert pourra par exemple payer des frais élevés pour que sa transaction soit sélectionnée avant celle de Charlie. Voir cette vidéo sur la mise en pratique de [front-run](https://youtube.com/watch?v=UZ-NNd6yjFM) (pour les impatients, [ici](https://www.youtube.com/watch?v=UZ-NNd6yjFM&t=955s)).

# L'approche retenue : le MEV
Alors comment faire ? Il y a plusieurs possibilités, et Madmat sur le discord Luchadores fait une excellente proposition ; passer par le [_mev_](https://ethereum.org/en/developers/docs/mev/) (_Miner/Maximum Extractable Value_). Sans rentrer dans les détails (voir ressources en fin d'article), il est possible d'envoyer des transactions directement à des mineurs, sans passer par le mempool.  
Et par ce moyen il existe une fonctionalité qui nous intéresse particulièrement ; envoyer un ensemble de transactions dans un même paquet, ce qui garantit qu'elles seront traitées ensemble, dans le même bloc. Par défaut si une transaction échoue le paquet entier est rejeté.    

Dans notre cas, nous allons forger deux transactions :
- Alice, la sponsor, envoie à Charlie (adresse compromise) de quoi payer le gas
- Charlie envoie son NFT à Bob (adresse propre)

Il nous faudra donc signer ces deux transactions avec respectivement les clés privées d'Alice et de Charlie.

# Mise en œuvre
L'équipe [Flashbots](https://docs.flashbots.net/) a publié un outil particulièrement adapté à notre problème : [sponsored-tx](https://github.com/flashbots/searcher-sponsored-tx)  
"_The use case for this multi-transaction setup is to make calls from an account that has a compromised private key._"  
Nous aurons besoin de ces scripts, de `npm`, et valoriser de manière appropriée les variables d'environnement suivantes :

```
PRIVATE_KEY_EXECUTOR     # clé privée compromise de Charlie
PRIVATE_KEY_SPONSOR      # clé privée d'Alice
RECIPIENT                # adresse publique de Bob sécurisée
FLASHBOTS_SECRET         # clé privée de réputation (v. explications + bas)
INFURA_API_KEY           # si nous passons par les nodes Infura
```
`FLASHBOTS_SECRET` est la clé privée d'une adresse servant uniquement à maintenir une [réputation](https://docs.flashbots.net/flashbots-auction/searchers/advanced/reputation) auprès des mineurs. Pour notre cas, nous aurons besoin de créer une adresse uniquement pour cette opération (voir ressources).  

## Avertissements
* **La nouvelle clé privée de Bob ne doit pas être divulguée.**
* **Ne jamais donner sa clé privée ni sa seed/mnemonic/passphrase. Jamais.**
* **Le fait de demander la clé privée d'un utilisateur est passible -à juste titre- d'un bannissement de tout endroit public qui se respecte.**
* **Posséder une clé privée est une responsabilité qui ne doit jamais être prise à la légère.** 

## Scripts
Après avoir récupéré et initialisé le projet 
```
git clone https://github.com/flashbots/searcher-sponsored-tx
cd searcher-sponsored-tx/src 
npm install
```  
Parcourir le readme, `index.ts` et `engine/*ts`.  
Note : `./contracts` ne sera pas utilisé.   
### Adapter et tester sur le testnet Görli
Créer un engine en s'inspirant de des fichiers engine/Approval721.ts et engine/TransferERC20.ts.  
Le cœur de la modification est la construction de la 2ème transaction, le `transferFrom()` (en mode quick win) :
```
...(await this._contractAddress721.populateTransaction.transferFrom(
	this._sender, this._recipient, this._tokenId))
``` 
Adapter en conséquence `index.ts` (import de l'engine approprié, provider : voir plus bas, adresse du contrat, ajout du tokenId).
### Connexions
Lignes 45 à 55 de `index.ts`.  
Soit nous utilisons le client [geth modifié](https://github.com/flashbots/mev-geth) que je n'ai pas utilisé (il semble falloir l'utiliser en syncmode full), soit via Infura, qui fonctionnera dans mon cas sur le mainnet. Dans le détail, c'est mev-geth qui introduit la possibilité d'envoyer un tableau de transactions. Plus d'explications [ici](https://docs.flashbots.net/flashbots-auction/searchers/advanced/rpc-endpoint).  
#### Configuration pour Görli
```
  const provider = new providers.InfuraProvider(5, process.env.INFURA_API_KEY || '');
  const flashbotsProvider = await FlashbotsBundleProvider.create(provider, walletRelay, 'https://relay-goerli.flashbots.net/');
```
'https://relay-goerli.epheph.com/' proposé par défaut n'avait plus de certificat valide lors de mes essais, mais a été corrigé depuis.  
#### Configuration pour mainnet
```
  const provider = new providers.InfuraProvider(1, process.env.INFURA_API_KEY || '');
  const flashbotsProvider = await FlashbotsBundleProvider.create(provider, walletRelay, 'https://relay.flashbots.net/');
```
## Tester
```
$ npm run start                                                                                                                                                               [10:38:09]

> @flashbots/searcher-sponsored-tx@0.0.1 start /tmp/searcher-sponsored-tx.bak
> npx ts-node --project tsconfig.json src/index.ts

⨯ Unable to compile TypeScript:
src/index.ts:46:9 - error TS2451: Cannot redeclare block-scoped variable 'flashbotsProvider'.
[...]
$ vi index.ts
$ npm run start 
[...]
[2021-12-15T10:20:50.006Z] Start!                                                                                                                                                            
[2021-12-15T10:20:50.052Z] (node:128445) UnhandledPromiseRejectionWarning: Error: missing argument: passed to contract (count=2, expectedCount=3, code=MISSING_ARGUMENT, version=contracts/5.
4.1) 
[...]
$ vi engine/TransferNFT.ts
$ npm run start
[...]
[2021-12-15T10:27:07.142Z] err: insufficient funds for gas * price + value: address 0x1C11E29C356A01724617C7b7e6C8f1DB713fBc37 have 0 want 651000000147000; txhash 0xe3c8e5b8bbaf43e6e3df7cf6
b22398c76dd31dea3b3288c21911b2affb38ab66
[2021-12-15T10:27:07.158Z] (node:128490) UnhandledPromiseRejectionWarning: Error: Failed to simulate response
[...]
# Get Goerli Faucet...
$ vi * ; npm run start # Vous voyez l'idée ; ne vous découragez pas vous touchez au but... 
[2021-12-15T10:34:13.439Z] Current Block Number: 6025888,   Target Block Number:6025890,   gasPrice: 31 gwei
[2021-12-15T10:34:29.125Z] Congrats, included in 6025889
```
![]({{site.baseurl}}/assets/img/lucha_tests.png)  
Une fois bien testé sur Görli, passer au mainnet.  

## Sauvetage !
Arranger les variables d'environnement, l'adresse du contrat, les adresses de connexion, le tokenId, assigner à PRIORITY_GAS_PRICE un prix en rapport avec le priority fee du moment (par exemple sur [blocknative](https://www.blocknative.com/gas-estimator)).   
**Note** : N'ayant pas d'expérience sur le sujet j'ai mis 31 gweis, soit 20 fois plus que le priority fee du moment, et mon bundle est passé au 6ème bloc. Voir le [readme](https://github.com/flashbots/searcher-sponsored-tx#setting-miner-reward).   

Et Go !
```
npm run start  # 🤞
```
```
[2021-12-15T11:27:46.629Z] Transfer 0xC899770c773Bc515Aa46a547CEaB49d796665725 approval for: [object Object]
[2021-12-15T11:27:46.629Z] Executor Account: 0x558cBD38E0901d28f01A23db5C9896364E8b8aC1
[2021-12-15T11:27:46.629Z] Sponsor Account: 0x52434Cd9e4e4F965a20c8576841CbAAC4b2bA30e
[2021-12-15T11:27:46.629Z] Simulated Gas Price: 35.48 gwei
[2021-12-15T11:27:46.630Z] Gas Price: 82.09 gwei
[2021-12-15T11:27:46.630Z] Gas Used: 126166
[2021-12-15T11:27:48.037Z] Current Block Number: 13809448,   Target Block Number:13809450,   gasPrice: 34.04 gwei
[2021-12-15T11:28:08.034Z] Current Block Number: 13809449,   Target Block Number:13809451,   gasPrice: 40.04 gwei
[2021-12-15T11:28:31.545Z] Not included in 13809450
[2021-12-15T11:28:32.192Z] Current Block Number: 13809450,   Target Block Number:13809452,   gasPrice: 34.79 gwei
[2021-12-15T11:28:39.547Z] Not included in 13809451
[2021-12-15T11:28:40.063Z] Current Block Number: 13809451,   Target Block Number:13809453,   gasPrice: 29.74 gwei
[2021-12-15T11:28:59.566Z] Not included in 13809452
[2021-12-15T11:29:00.065Z] Current Block Number: 13809452,   Target Block Number:13809454,   gasPrice: 30.26 gwei
[2021-12-15T11:29:03.576Z] Congrats, included in 13809453
```
![]({{site.baseurl}}/assets/img/lucha_mev.png)  
![]({{site.baseurl}}/assets/img/green_beret.png)  
À approfondir : l'estimation du gas pour ne pas laisser d'Ether dans l'adresse compromise.  

# Ressources

## MEV
[Définition](https://ethereum.org/en/developers/docs/mev/)  
### Projet Flashbots
[Projet Flashbots](https://docs.flashbots.net/), [ressources](https://docs.flashbots.net/new-to-mev), [Git Flashbots](https://github.com/flashbots)   
Contacter [leur service whitehat](https://docs.flashbots.net/whitehat) pour se faire aider à récupérer les fonds d'un wallet compromis.     
[How to use Flashbots](https://cryptomarketpool.com/how-to-use-flashbots/)  
[Dashboard MEV](https://explore.flashbots.net/)

## Faucets Görli
* ERC20 : envoyer 0 ether vers [ce contrat](https://goerli.etherscan.io/address/0xaff4481d10270f50f203e0763e2597776068cbc5#code) pour recevoir des tokens Weenus
* ERC721 : `mint(quantity)` max 3 dans [ce contrat](https://goerli.etherscan.io/address/0xd000f000aa1f8accbd5815056ea32a54777b2fc4#writeContract) pour des TestToadz

## Créer des adresses avec python
{% highlight python %}
from eth_account import Account
import secrets
priv = secrets.token_hex(32)
private_key = "0x" + priv
print ("SAVE BUT DO NOT SHARE THIS:", private_key)
acct = Account.from_key(private_key) # <--- permet aussi de vérifier une adresse publique
print("Address:", acct.address)
{% endhighlight %}
Source :[How to generate a new Ethereum address in Python](https://www.quicknode.com/guides/web3-sdks/how-to-generate-a-new-ethereum-address-in-python)


## Divers
Pour exemple, [mon adresse "compromise" de test](https://goerli.etherscan.io/address/0x41e0238a32945c316f7ecacf3a0f3ca2f78b4268).  
[Rush'n Attack](https://www.youtube.com/watch?v=STjMHqeF3R0)

-----
# Quick Start (EN)
## Scripts
```
git clone https://github.com/flashbots/searcher-sponsored-tx
cd searcher-sponsored-tx/src
npm install
more ../README.md
```
```
vi index.ts
* Lines 9,10  : import { Approval721 } from "./engine/Approval721"; 
  import the engine you'll create or modify
* Line 17     : const PRIORITY_GAS_PRICE = GWEI.mul(31)
  set PRIORITY_GAS_PRICE as needed
* Lines 44-55 : const provider = new providers.InfuraProvider(5, process.env.INFURA_API_KEY || '');
  set up the connection parameters
* Lines 61-71 : const tokenAddress = ...
  modify the contract address
```
Create or modify the imported engine according to your needs in `./engine/` ; adapt it to your particular requirements. In our case the main modifications were 
```
...(await this._contractAddress721.populateTransaction.transferFrom(
	this._sender, this._recipient, this._tokenId))
```
## Environment variables
Create a new random address for FLASHBOTS_SECRET asked by miners. A one shot address in our case.
```
export PRIVATE_KEY_EXECUTOR     # compromised private key
export PRIVATE_KEY_SPONSOR      # sponsor's private key
export RECIPIENT                # sane public key
export FLASHBOTS_SECRET         # new private key 
export INFURA_API_KEY           # if we use a Infura node
```

