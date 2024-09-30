# Physique

## Mise en place

Un des "hello world" de l'animation est celui de la balle qui rebondit sur les bords de l'écran. 
Nous allons réaliser la chose en ajoutant une simulation de la gravité.
Nous allons utiliser un modèle simple permettant toutefois d'obtenir un bon résultat.
Ce compromis fait que c'est une solution souvent utilisé pour une simulation physique performante.

## Class CircleVerlet
Créez un nouveau fichier nommé *CircleVerlet.js* et codez une classe héritant de la classe *Circle*.
Dans cette nouvelle classe, nous allons étendre la classe de base. Ajouter un paramètre *dt* au constructeur.
Appelez le constructeur de la classe parente, puis stocker x et y comme prosition précédente du cercle.
Enfin, inversez la direction et appelez la méthode *move* de la classe parente avec ce *dt* mais négatif.
Cela permettra ainsi d'avoir une position courant et la position précédente nécessaire a l'intégration de Verlet.

### Méthode *move*
Surcharger la méthode *move* de la classe parente pour calculer les (x,y) après un Δt. 
Pour ce faire l'intégration de Verlet nous propose la formule suivante:
```js
const velocityX = (currentX - lastX);
lastX = currentX;
currentX = currentX + velocityX + accelX * dt * dt;
```
Où lastX est la position X précédente du cercle et accelX l'accélération sur l'axe X.
Pour le moment mettez votre accélération à 0 et procéder de la meme manière pour les y.

### Boucle d'animation
Dans votre programme principal, testez votre nouvelle classe en créant un cercle. 
Puis créez une *boucle* d'animation en utilisant [MainLoop.js](https://github.com/Chabloz/devmobil51/blob/main/src/utils/mainloop.js), et faites bouger votre cercle. 

### Appliquer la gravité
Rajoutez une méthode *applyForceY* dans votre classe. Cette méthode recevra une force (la "gravité" dans notre cas) et modifiera la propriété *accelY* en conséquence.
Modifierz votre méthode move pour remettre à 0 la valeur de accelY après le mouvement.
Enfin, appliquez la gravité à chaque "tick" de votre simulation.

### Méthode *boxConstraint*
Rajoutez une méthode *boxConstraint* dans votre classe prenant une largeur et hauteur. 
Lorsque le cercle touche (ou déborde) des bords de la boite, remettez la à une position valide. 
Toutefois, changer la dernière position du cercle comme à celle que le cercle aurait prise si il avait continué à travers le "mur". 
Appelez cette nouvelle méthode dans votre boucle d'animation juste après le mouvement de votre cercle pour la tester.

### Gestion des interactions
Mettez en place une interaction simple. Lors d'un click de souris, récupérez sa position.
Recherchez alors dans la collection de cercles le premier cerle en colision avec ce point.
Pour ce faire vous pouvez premièrement calculer la distance entre la position de la souris et le centre du cercle:

```js
distanceTo({x , y}) {
  const dx = this.x - x;
  const dy = this.y - y;
  return Math.sqrt(dx*dx + dy*dy); // Pythagore
}
```

Puis, vérifiez si cette distance est inférieure au rayon (dans ce cas le point est dans le cercle):

```js
isInside({x, y}) {
  return this.distanceTo({x, y}) < this.r;
}
```

Enfin, appliquez un force sur le cercle en question. 
