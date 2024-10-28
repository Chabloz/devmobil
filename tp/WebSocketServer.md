# WebSocket

### Objectif
Prendre en main la technologie WebSocket.

### Mise en place
Commencez par installer [`ws`](https://github.com/websockets/ws/tree/master) dans votre projet grâce à `npm` avec la commande suivante :
```
npm install ws
```
Pour organiser votre projet et faciliter le démarrage du serveur WebSocket, vous pourriez:
Créez un dossier `backend` à la racine de votre projet
Créez un fichier `wsServer.js` dans ce dossier pour y placer le code du serveur WebSocket. Pour tester, vous pouvez ajouter la ligne suivante dans ce fichier :
```js
console.log('Hello World!');
```
Ensuite, modifiez le fichier `package.json` pour ajouter une commande `npm` afin de lancer le serveur (il devrait déjà y avoir des commandes dans la partie `scripts`) :
```js
{
  "scripts": {
    "server": "node backend/wsServer.js"
  }
}
 ```

Enfin, lancez le serveur WebSocket avec la commande suivante :
```
npm run start
```
Si tout ce passe bien, `Hello` devrait s'afficher dans votre console.

## WebSocket Server

Créez une classe `WebSocketServerOrigin` en étendant la classe `WebSocketServer` du package `ws`. Le but est d'y rajouter les fonctionnalités suivantes:

**Nouvelles options au constructeur** :  Pour améliorer la sécurité, le constructeur pourrait prendre en paramètre une liste d'origines autorisées. Cela permettra de filtrer les connexions en fonction de l'origine de la requête.
De plus, vous pouvez ajouter une option pour définir le nombre maximal de connexions simultanées autorisées afin de limiter la charge du serveur.

**Amélioration du handshake** :
La méthode `handleUpgrade` (qui est celle qui s'occupe du *handshake*) vérifiera l'origine des requêtes entrantes. Les requêtes avec une origine non autorisée seront rejetées avec un code HTTP 403 (Forbidden).
La méthode `handleUpgrade` testera aussi si le nombre maximal de connexions simultanées est atteint. Si c'est le cas, la connexion sera refusée avec un code HTTP 503 (Service Unavailable).
Voici un exemple de code pour la méthode `handleUpgrade` ( checkOrigin est une méthode à rajouter par vos soins) :

```js
handleUpgrade(request, socket, head, callback) {
  if (!this.checkOrigin(request.headers?.origin)) {
    socket.write('HTTP/1.1 403 Forbidden\r\n\r\n');
    socket.destroy();
    return;
  }
  if (this.clients.size >= this.options.maxNbOfClients) {
    socket.write('HTTP/1.1 503 Service Unavailable\r\n\r\n');
    socket.destroy();
    return
  }

  return super.handleUpgrade(request, socket, head, callback);
}
```

### Encapsulation de WebSocketServerOrigin

Pour encapsuler la classe `WebSocketServerOrigin` et y ajouter des fonctionnalités propres à un projet, créez une nouvelle classe `WSServer`.
Le constructeur de cette classe prendra en paramètre les options du serveur WebSocket. Voici une proposition:

```js
{
  port = 8887,
  maxNbOfClients = 30,
  verbose = true,
  origins = 'http://localhost:5173',
  pingTimeout = 30000,
}
```

Une méthode start permettra de lancer le serveur WebSocket. Elle écoutera les événements de connexion des clients et appelera une méthode `onConnection` pour gérer les connexions entrantes.
Elle s'occupera aussi de gérer les fréquences de ping pour vérifier que les clients sont toujours connectés.

La méthode onConnection prendra en paramètre un objet `client`.  Elle s'occupera de gérer les événements de message, de fermeture et de pong des clients. Voici un exemple de code pour cette méthode :

```js
  const clientId = this.createClientMetadata(client);
  this.log(`Client ${clientId} connected`);
  client.on('message', message => this.onMessage(client, message));
  client.on('close', () => this.onClose(client));
  client.on('pong', () => this.onPong(client));
```
La méthode `createClientMetadata` permettra de stocker des informations sur les clients connectés. Pour le moment elle ne fera qu'attribuer un identifiant unique à chaque client et un status isAlive à true pour la gestion des pings/pongs. Vous pourriez utiliser une structure de données Map pour stocker ces informations pour chaque client.
Il vous reste à implémenter les méthodes `onMessage`, `onClose` et `onPong` pour gérer les messages, les fermetures de connexion et les pings/pongs des clients. Pour `onMessage`, tous les messages reçus seront *broadcastés* à tous les clients sauf l'expéditeur.
Pour simplifier la gestion des envois des messages, vous pouvez ajouter les méthodes `send`, `broadcast` et `broadcastToOthers` à la classe `WSServer`. Vérifiez bien que le client à un `readyState` égal à `WebSocket.OPEN` avant de lui envoyer un message. Il pourrait être pratique que ces méthodes convertissent automatiquement les données en JSON avant de les envoyer.

## WebSocket: PubSub et RPC
Une fois cette première prise en main faites, je vous conseille d'utiliser un code plus architecturé. Ce que vous avez fait jusqu'à présent devrait vous aidez à comprendre facilement la proposition suivante.

Pour architecturer votre système de communication WebSocket, l’usage du Design Pattern PubSub (Publisher-Subscriber) et de RPC (Remote Procedure Call) est efficace et couramment implémenté. Le PubSub permet aux clients de publier des messages dans des canaux spécifiques et de s'y abonner pour recevoir des messages. Le RPC offre la possibilité d’appeler facilement des fonctions du WebSocket serveur.

Vous trouverez le code d'un serveur et d'un client ici : [code source PubSub et RPC](https://github.com/Chabloz/devmobil51/tree/main/src/websocket). Reprenez l'entierté du dossier et ajoutez le dans votre dossier *src*. 

### Documentation succinte pour la partie serveur

La classe WSServerPubSub permet de créer un serveur WebSocket. Elle accepte une fonction *authCallback* pour authentifier et associer des métadonnées aux clients: *authCallback(token, request, wsServer)*. Cette fonction est appelée pour chaque connexion entrante. Si cette fonction retourne *false*, la connexion est refusée. Sinon, elle permet d'ajouter des métadonnées personnalisées en retournant un objet les contenants. Exemple:

```js
function authCallback(token, request, wsServer) {
  // Use the token here for the authentification of the client
  // You must return false if access is denied.
  return {username: 'Anonymous'};
}

const wsServer = new WSServerPubSub({
  port: 8887,
  origins: 'http://localhost:5173',
  authCallback,
});
```

La méthode `addChannel` permet d'ajouter un canal de communication au serveur avec des options personnalisables :

- **`chan`** : nom du canal.
- **`options`** : 
  - **`usersCanPub`** *(boolean, défaut : `true`)* : spécifie si les utilisateurs peuvent publier sur ce canal.
  - **`usersCanSub`** *(boolean, défaut : `true`)* : spécifie si les utilisateurs peuvent s’abonner.
  - **`hookPub`** *(fonction)* : callback exécuté avant la publication d’un message. Doit retourner le message après modifications éventuelles. Appelée avec le message, les métadonnées du client et le serveur.
  - **`hookSub`** *(fonction)* : callback exécuté avant qu'un client s'abonne, doit renvoyer `true` pour autoriser ou `false` pour refuser l'abonnement.
  - **`hookUnsub`** *(fonction)* : callback exécuté avant qu'un client se désabonne, sans impact sur l’action de désabonnement.

Exemple :
```js
wsServer.addChannel('chat', {
  usersCanPub: true,
  usersCanSub: true,
  hookPub: (msg, client, wsServer) => {
    return {...msg, from: client.username, time: Date.now()};
  },
});
```

La méthode `addRpc` permet de définir une fonction RPC accessible par les clients :

- **`name`** : nom de la RPC.
- **`callback`** : fonction exécutée lors de l’appel, qui doit renvoyer une réponse. Reçoit les données envoyées par le client, les métadonnées du client, et le serveur. Elle peut lever une erreur `WSServerError` pour indiquer une erreur au client.

Exemple :
```js
wsServer.addRpc('hello', (data, client, wsServer) => {
  if (!data?.name) throw new WSServerError('Name is required');
  return `Hello from WS server ${data.name}`;
});
```

### Documentation succinte de la partie cliente

La classe `WSClient` permet de gérer la connexion WebSocket côté client et de simplifier l’utilisation des fonctionnalités de PubSub et de RPC.

- **`connect()`** : Établit la connexion au serveur WebSocket. Renvoie une promesse qui se résout une fois la connexion établie. En cas d'échec, la promesse est rejetée avec une erreur. Il est possible de passer un token d'authentification à la méthode. Il sera reçu dans la fonction *authCallback* du server pour la gestion de l'autorisation.

- **`sub(channel, callback)`** : Permet de s'abonner à un canal de messages (par ex. `"chat"`). Le **`callback`** est une fonction qui sera exécutée pour chaque message reçu sur ce canal. Plusieurs abonnements peuvent être créés pour un même canal. Renvoie une promesse qui se résout lorsque la subscription est réussie, ou est rejetée en cas d’erreur.

- **`pub(channel, message)`** : Publie un message dans un canal. Renvoie une promesse qui se résout lorsque le message est envoyé, ou est rejetée en cas d’erreur.

- **`rpc(method, params)`** : Appelle une méthode RPC sur le serveur. **`method`** est le nom de la méthode à appeler (par ex. `"hello"`), et **`params`** contient les paramètres envoyés avec l'appel. Renvoie une promesse qui se résout avec la réponse du serveur ou est rejetée en cas d'erreur.

## WebSocket Client
Pour tester le serveur, mettez en place un chat simple avec un client WebSocket.
Puisque les données sont réactives, vous pouvez utiliser un framework comme Vue.js pour gérer l'affichage des messages. Vous pouvez vous baser sur ce qui a été fait l'année dernière pour le chat en ligne ( [ici](https://chabloz.eu/webmobui/exercices)). Pour le moment ne gérer pas les noms d'utilisateurs et les gens connectés (laisser n'importe qui envoyer des messages). Vous pouvez vous aider de la documentation de `WebSocket` de MDN pour mettre en place le client ([MDN WebSocket Client](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications)). Essayez d'architecturer au mieux votre application. N'hesitez pas à venir me voir si vous avez des questions sur votre architecture.

Si vous partez sur les mêmes technlogies que l'année passée, voici une petite doc pour leur installation:

### installation de Vue
```bash
npm install vue@latest
npm install @vitejs/plugin-vue
```

### installation de Quasar (optionnel)
```bash
npm install --save quasar @quasar/extras
npm install --save-dev @quasar/vite-plugin
```
Modifiez le fichier vite.config.js avec *Vue seulement:*
```javascript
import vue from '@vitejs/plugin-vue';

// et dans la partie plugin:
plugins: [vue()],
```

*Vue + Quasar:*
```javascript
import vue from '@vitejs/plugin-vue';
import { quasar, transformAssetUrls } from '@quasar/vite-plugin';

// et  dans la partie plugin:
plugins: [
  vue({template: { transformAssetUrls }}),
  quasar(),
],
```

