# DMX Controller

Ce programme est un serveur ecrit en NodeJS qui permet de controller de multiples appareils DMX via différents client (Web et Android).

Ici, il s'agit du serveur. Ce composant est a la fois le serveur auquel les clients vont se connecter et le proxy sur lequel les différents appareils DMX vont s'enregistrer et recevoir leurs comandes respectuves.
Le DMX etant un proticole de gestion d'appareils lumineux, il est necessaire que le client et l'appareil soient connectes en temps réel. Par conséquent, l'utilisation d'un socket est necessaire. Nous utiliserons la librairie SocketIO pour l'implémenter.

## Idée

Les appareils utilisent le logiciel QLC+ et sont des Raspberry Pi Zero.
L'implementation du socket est donc dependante de ce logiciel. Par conséquent, seules certaines fonctionnalités sont disponibles et une partie du travail sera a gerer côté serveur.
Nous disposons de fonction permettant de controller les Channels une par une, de récupérer leurs valeurs, de selectionner le controleur USB/DMX. Il est en projet de ne plus avoir besoin de ce logiciel, et d'utiliser les ports GPIO de ces appareils, mais cela sera reserve a la version 2.0
Ce serveur doit pouvoir enregistrer des appareils DMX, or, ces derniers seront connectes via un reseau 4G vraisemblablement, afin d'assurer leur mobilite. Ne disposant pas d'ip fixe, nous nous tournerons vers un tunnel ssh vers un des serveurs d'enregistrement, qui eux possedent une IP fixe. On doit donc rejouter le ping dans les delais de transmission. 
Cependant, les WebSockets envoient des messages legers, et une connexion 2G ou 3G est suffisante pour acheminer l'information de maniere suffisamment efficace.

## Le(s) serveur(s) d'enregistrement

Ils permettront une utilisation de tous les ports pour des tunnels SSH ainsi que proposeont un acces SSH pour debugger.
Nous utiliseront aussi AutoSSH, afin de ne pas bloquer un port si l'un des appareil se deconnecte.
(Utilisation de SocketIO? le problème etnt que du coup il faudrait un transmetteur entre le web socket et l'application, puisqu'elle n'utilise pas socketIO, et que le choix du port n'est pas possible.)
Ils seront enregistres, avec leurs caracteristiques dans le serveur principal, qui servira de load-balancer, afin de repartir les appareils sur differents serveurs.


## Le serveur principal

C'est ce serveur qui repose dans ce GIT. Il doit permettre d'enregister un serveur d'enregistrement, et de communiquer entre les clients et les appareils DMX.

Pour un client, ce serveur doit permettre de creer/connecter des utilisateurs, de recuperer des valeurs de channels, de definir leur valeur, d'enregistrer des fixtures, d'enregistrer une interface graphique sur son compte utilisateur,  de lister ses appareils, se connecter a un appareil, et de recuperer la qualite de connexion de l'appareil.
Dautres fonctionnalites peuvent etre ajoutees en cours de creation (fonctions, timeline...).

L'objectif de la version 0.1 est de prodiguer la possibilité de contrôler un appareil DMX seul, via une interface Web) via des sliders et des boutons.

Dans la version 0.2, il sera possible de creer des comptes utilisateurs et d'enregistrer des fixtures.

Nous avons vu que nous allions utiliser SocketIO pour ces transferts de donnees.
Precisons que les channels sont indicees entre 0 et 511, mais dans les vues, l'indiçage commencera bien évidemment à 1.
Voici les évenements auquels le serveur pourra repondre:

	- "getChannels" :
		- Permet de récupérer les données d'un ensemble de channels
		- INPUT:
  			channels : Array<int> correspondant aux indices (donc de 0 à 511) dont on veut récupérer les valeurs
		- OUTPUT: Objet JavaScript avec pour donnees:
  			values: Array<Object> chaque objet ayant une clé index et value.

  	- "getAllChannels" :
		- Permet de récupérer l'intégralité des channels avec leurs valeurs
		- INPUT: aucun
		- OUTPUT: Array<Object>(512) chaque objet representant une channel avec son index, et sa valeur.

  	- "setChannel" :
		- Permet de définir la valeur d'une channel.
		- INPUT:
		    index: int - index de la channel à modifier (entre 0 et 511)
            value: int - valeur à attribuer (entre 0 et 255)
		- OUTPUT: aucun

La suite viendra ultérieurement, de nombreuses discussions sont en cours sur la gestion des entrées et sorties.