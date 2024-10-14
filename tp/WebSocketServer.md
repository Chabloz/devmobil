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


## WebSocket Client
Actuellement le serveur WebSocket est prêt à recevoir des connexions et à faire des *broadcasts* de messages. Pour tester le serveur, mettez en place un chat simple avec un client WebSocket.
Puisque les données sont réactives, vous pouvez utiliser un framework comme Vue.js pour gérer l'affichage des messages. Vous pouvez vous baser sur ce qui a été fait l'année dernière pour le chat en ligne ( [ici](https://chabloz.eu/webmobui/exercices)). Pour le moment ne gérer pas les noms d'utilisateurs et les gens connectés (laisser n'importe qui envoyer des messages). Vous pouvez vous aider de la documentation de `WebSocket` de MDN pour mettre en place le client ([MDN WebSocket Client](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications)). Essayez d'architecturer au mieux votre application. N'hesitez pas à venir me voir si vous avez des questions sur votre architecture.
