---
layout: post
title:  "Configurer une node de test merge eth2"
date:   2021-12-06 01:00:00 +0200
categories: ethereum node docker
---

![](https://github.com/sznicolas/protoSvgLib/raw/main/img/logoEth.svg){: width="64" }
Tester le merge avec Linux (Ubuntu) et Docker.

# Prérequis 
* Une machine juste décente, <1Go d'espace disque
* Metamask avec une adresse de test, inutilisée sur le mainnet.
Attention ; il se peut que cela ne fonctionne pas sous Windows et Mac à cause de l'option docker-compose `network_mode: host`. Non testé.   
Cf [Documentation Docker](https://docs.docker.com/network/host/) 

# Installer docker + docker-compose
<details markdown="1">
<summary>Détails...</summary>
## Docker
```
sudo apt-get update
 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

```
[Doc officielle](https://docs.docker.com/engine/install/ubuntu/)
## Docker compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# si besoin :
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
[Doc officielle](https://docs.docker.com/compose/install/)

## Droits
Seul root et les utilisateurs appartenant au groupe `docker` peuvent exécuter des commandes docker.
```
sudo usermod -aG docker nom_du_user
```
Puis reconnecter l'utilisateur.
</details>

# Configurer et lancer les nodes
## Récupérer et configurer l'environnement
Depuis votre répertoire de travail, lancer les commandes suivantes :
```
git clone https://github.com/parithosh/consensus-deployment-ansible 
cd consensus-deployment-ansible/scripts/quick-run/merge-devnets
mkdir -p execution_data beacon_data
```
Éditer `merge-devnet-2.vars` pour y mettre son IP externe en ligne 2, `IP_ADDRESS=`.  
Votre adresse visible depuis internet peut être récupérée comme suit :
```
curl ifconfig.me
```
Ou se rendre sur ce [site](https://whatismyipaddress.com/)

## Lancer les nodes
Plusieurs _execution engines_ sont disponibles, nous utiliserons `geth` dans cet exemple.
```
docker-compose --env-file merge-devnet-2.vars -f docker-compose.geth.yml up -d
```
Plusieurs _consensus engines_ sont disponibles, nous utiliserons `lighthouse` dans cet exemple.
```
docker-compose --env-file merge-devnet-2.vars -f docker-compose.lighthouse.yml up -d
```
## Vérifier les logs
```
docker logs lighthouse_beacon -f --tail=20
docker logs geth -f --tail=20
```
Les logs des nodes synchronisées devraient respectivement ressembler à :
```
Dec 06 14:30:12.258 INFO New block received                      root: 0x19506f6b601f5e82a5948ee1092dc881690d9eca878acedab560edf680cd59ab, slot: 42726
Dec 06 14:30:18.000 INFO Synced                                  slot: 42726, block: 0x1950…59ab, epoch: 1335, finalized_epoch: 1333, finalized_root: 0x7968…ad41, exec_hash: 0xd0c3…9072 (verified), peers: 4, service: slot_notifier
```

```
INFO [12-06|14:26:36.162] Setting head                             head=f38846..e837e5
INFO [12-06|14:26:36.162] Set the chain head                       number=39101 hash=f38846..e837e5
```

Il faut patienter le temps que les nodes se synchronisent, nous pouvons observer les derniers numéros de block sur cet [explorateur](https://explorer.merge-devnet-2.wenmerge.dev/blocks)

[Source](https://hackmd.io/dFzKxB3ISWO8juUqPpJFfw)

# Configuration de Metamask

| **Network Name** | Devnet-2              |
| **New RPC URL**  | http://127.0.0.1:8545 |
| **Chain ID**     | 1337502               |

[Source](https://hackmd.io/dFzKxB3ISWO8juUqPpJFfw#Setting-up-Metamask)  

Astuce : si la machine hébergeant docker n'est pas la machine depuis laquelle nous utilisons Metamask, ouvrir un tunnel depuis le poste de travail :
```
ssh user@serveur_docker -v -N -L 8545:127.0.0.1:8545
```
Merci à canbo pour cette astuce.  
Attention, il a été reporté un problème empêchant de finaliser la configuration dans certains cas. `Could not fetch chain ID. Is your RPC URL correct?`  
Mes essais montrent que :
Metamask 10.1.0 dans Firefox == KO  
Metamask 10.6.4 dans Brave == Ok

## Récupérer des ethers de test
Sur le [faucet](https://faucet.merge-devnet-2.wenmerge.dev/).  
**N'Utiliser Que des Adresses de Test.**
Tester l'envoi d'ethers.

# Staking
Pour staker, il faut utiliser la fonction `Deposit()` du [DepositContract](https://explorer.merge-devnet-2.wenmerge.dev/address/0x4242424242424242424242424242424242424242/transactions).
1. Se connecter au container Lighthouse
```
docker exec -it lighthouse_beacon bash
```
2. Créer un compte
```
lighthouse --testnet-dir=/custom_config_data account wallet create
```
Suivre la procédure, bien conserver les informations fournies (wallet name, mot de passe saisi, seed phrase, UUID).

3. Générer les données du validateur
```
lighthouse --testnet-dir=/custom_config_data account validator create --count 1
```

Suivre la procédure :

```
[...]
Enter wallet name:
mon_wallet

Enter your wallet's password: mon_password

1/1	0x940e9c65ca3c7fe17ea8977994aeb5a336a545b4197e67612c14ac6c4957ace2e43c1353d19d1ab6780705094189a0d2
```
La chaîne `0x940...a0d2` correspond à mon numéro unique de validateur. Les données dont nous avons besoin pour continuer se trouvent dans le fichier `eth1-deposit-data.rlp`. Adapter et lancer la commande suivante :
```
cat /root/.lighthouse/custom/validators/0x940e*/eth1-deposit-data.rlp ; echo
```
Et récupérer le résultat qui ressemble à ceci (c'est une information publique) :
```
0x22895118000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000000e0000000000000000000000000000000000000000000000000000000000000012093dfec8329de953f5fb4ccb88bfe22318da461d17d17176f79239c92814ee1880000000000000000000000000000000000000000000000000000000000000030940e9c65ca3c7fe17ea8977994aeb5a336a545b4197e67612c14ac6c4957ace2e43c1353d19d1ab6780705094189a0d2000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000020006727b930b61d84fee4feba93d0ffd01c067f35f0a2c73b901650756f6f3240000000000000000000000000000000000000000000000000000000000000006095b8dead6a5111e36258e0a88feb680dd74a5647947af8f7ebef1c27a3040d8bcf734b259b5946270b65d7e5f32211ff11ae86f0645e148212355dc1bb1090deb16d1373b8465732a302e16381718eb821c9da9c236f225862f0a7b9bcf32a85
```
Cela contient, encodé, l'appel de fonction `Deposit()` avec les paramètres qui nous concernent pour activer le smart contract.  
Nous pouvons maintenant envoyer 32 Ethers au smartcontract 0x424242... , en ajoutant dans le champ HexData la longue ligne du fichier eth1-deposit-data.rlp copiée depuis le terminal.
![]({{site.baseurl}}/assets/img/merge-metamask-data.png)

## Suivi
Vous devriez voir votre transaction [dans l'explorateur de blocks](https://explorer.merge-devnet-2.wenmerge.dev/address/0x4242424242424242424242424242424242424242/transactions).  
Coller son numéro de validateur (0x940... dans mon exemple) dans le [Beacon Chain Explorer](https://beaconchain.merge-devnet-2.wenmerge.dev/). Rapidement nous devrions voir notre entrée, mais, contrairement à ce qui est indiqué, il faudra plusieurs heures pour être un validateur validé ;-)
![]({{site.baseurl}}/assets/img/merge-beacon-validator.png)  
