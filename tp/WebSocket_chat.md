# Contexte

Réaliser un **chat** en utilisant **WebSockets**. Vous devez mettre en place le backend (avec la lib websocket **wsmini**  et Node.js) et le frontend avec **Vue**. Le chat permettra à plusieurs utilisateurs de discuter dans un canal unique, de voir la liste des participants connectés et d'utiliser quelques commandes (préfixée par /).

## Objectifs pédagogiques

- Prendre en main les WebSockets via la mise en place d'un serveur    
- S'initier au Publish Subscribe (PubSub) via la gestion de plusieurs canaux 
- Gérer l'échange des messages entre plusieurs clients et mettre à jour l’interface en temps réel.    
- S'initier au Remote Procedure Call (RPC) via l'implémentation d'un système de commandes.    

## Installation des dépendances

En partant de votre projet existant ou d'un nouveau projet Vite + Vue : 
1.  Installer [WsMini](https://github.com/Chabloz/WsMini) en consultant la doc de la lib
2. Créez un fichier, par exemple *wsChatServer.mjs*, à la racine du projet. Il contiendra la logique du serveur WebSocket basé sur WsMini.
3. Ajoutez dans `package.json` un script pour démarrer votre serveur WS , Vous pourrez ainsi lancer le serveur avec `npm run ws-chat`.
```js
"scripts": {
  "ws-chat": "node wsChatServer.mjs"
}
```

## Mise en place du serveur WebSocket

Le serveur reposera sur la classe [WSServerPubSub](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSServerPubSub.md). Cette classe fournit un système de **canaux (channels)** pour publier/recevoir des messages ainsi qu’un système de **RPC** pour exécuter des commandes. Le serveur doit gérer les points suivants:
**Remarques**:  développer la chose étape par étape  conjointement avec le coté client pour faciliter vos débugages.

### L’authentification 
L'authentification se fera via un **token** qui ne contiendra que le nom d’utilisateur. Comme nous n'auront pas de vrai système d'authentification, la seul chose a faire dans le [authCallback](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSServerPubSub.md#new-wsserverpubsuboptions), est de vérifier que ce nom d'utilisateur est composé de caractère alphabétique (expression régulière `/^[A-Za-z]+$/`) et qu’il ne dépasse pas une vingtaine de caractères. Profiter du *return* de *authCallback* pour ajouter des méta-data au client si la connexion est acceptée, comme par exemple une couleur aléatoire (qui sera utilisée pour l’affichage de son *username* quand il enverra des messages).

### Les canaux    
Le serveur fera la création de deux canaux : un canal `chat` pour les messages publics et un canal `users` pour envoyer la liste des utilisateurs connectés. Utilisez  la méthode [addChannel](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSServerPubSub.md#addchannelname-options) avec les bonnes options. Utilisez un *hook* (option `hookPub`) pour interdire les messages trop longs et rajouter les données nécessaires au message de l'utilisateur (comme son *username*, et un *timestamp*).

La liste des utilisateurs connectés sera publiée manuellement par le serveur à chaque changement via le canal `users` en utilisant la méthode [pub](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSServerPubSub.md#pubchanname-msg). Pour observer tous les changements dans la liste des utilisateurs, vous pouvez utiliser les *hooks* [hookSubPost et hookUnsubPost](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSServerPubSub.md#addchannelname-options) . Vous pouvez obtenir les metadata de touts les clients inscrit à un canal avec [getChannelClientsData](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSServerPubSub.md#getchannelclientsdatachanname).


    
### Implémentation des commandes via RPC

WsMini propose un mécanisme de **RPC** permettant d’exposer des fonctions que les clients peuvent appeler. Vous devez réaliser au minimum les deux commandes suivantes: 

-   **`/em <text>`** : diffuse un message de type `emote` à tous les clients du chat (le texte sera en italique par exemple).
    
-   **`/pm <username> <text>`** : envoie un message privé grâce à [sendCmd](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSServerPubSub.md#sendcmdclient-cmd-data) pour envoyer le message au bon destinataire.

## Intégration coté client

Le client devra se connecter au serveur WS, afficher les messages en direct, maintenir la liste des utilisateurs en ligne et envoyer des commandes RPC. Pour ce faire, utilisez la classe [WSClient](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSClient.md) de *WsMini*. N’instanciez qu’un seul `WSClient` par client et conservez‑le dans un état global (dans un store par exemple) afin de le réutiliser dans les différentes parties de votre interface.

### Page de connexion

Créez un formulaire demandant un **username** (20 caractères max). Utilisez l’attribut `pattern="[A-Za-z]+"` pour la validation HTML ou un test regex en JS. Puis effectuez la connexion au serveur WS lors du *submit* tout en gérant correctement les erreurs si la promesse échoue.

### Gestion des canaux

 Si tout c'est bien passé, abonnez le client au deux canaux grâce à la méthode [sub](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSClient.md#subchan-callback-timeout). Et effectuez la mise à jour des structures réactives pour les messages du chat ainsi que la liste des utilisateurs connectés. Effectuez un scroll automatique vers le bas lors de la réception d'un nouveau message. 

L'utilisateur pourra envoyer des message sur le canal du chat. Utilisez la méthode [pub](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSClient.md#pubchan-msg-timeout) en gérant  correctement l’échec de la promesse.
 
### Envoi et gestion des commandes /

Lors de la soumission du formulaire de chat, vous devez détecter si le message commence par un **/**.
Si c'est le cas, il ne faut alors pas le publier dans le canal, mais executer la commande sur le serveur WS grâce à la méthode [rpc](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSClient.md#rpcname-data-timeout).
Il faudra donc aussi correctement gérer les retours des commandes (**em** et **pm**) que le serveur enverra.

### Déconnexion

Ajoutez un bouton de *logout* dans l’interface de chat. Lors du clic, fermer la connexion WS avec [close](https://github.com/Chabloz/WsMini/blob/main/docs/api/WSClient.md#close) puis rediriger l'utilisateur sur la page de login.

## Limitations et pistes d’amélioration

Le mini‑chat présenté ici est volontairement simple. Plusieurs aspects indispensables à un vrai service de chat n’ont pas été traités :

- **Authentification**: un vrai système d'authentification serait nécessaire.

-   **Modération et filtrage** : aucune vérification de spam, d’injures ou de flood n’est réalisée.
    
-   **Limitation du débit** : un utilisateur pourrait saturer le serveur en envoyant trop de messages. L’implémentation d’un *rate‑limiting* est à envisager.
    
-   **Gestion des caractères spéciaux et des accents** : les diacritiques peuvent poser problème (par exemple via des [générateurs de *glitch*](https://zalgo.org/) . 

-   **Historique des messages** : les nouveaux arrivants ne reçoivent pas les messages déjà échangés. Une sauvegarde en base de données serait nécessaire (et serait aussi utile pour la modération).    

Ces limites font que le chat actuel n’est pas encore prêt pour laproduction.


