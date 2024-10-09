# Physique

## Mise en place

Un des "hello world" de l'animation est celui de la balle qui rebondit sur les bords de l'écran.
Nous allons réaliser la chose en ajoutant une simulation de la gravité.
Nous allons utiliser un modèle simple permettant toutefois d'obtenir un bon résultat.
Ce compromis fait que c'est une solution souvent utilisé pour une simulation physique performante.

## Class CircleVerlet
Créez un nouveau fichier nommé *CircleVerlet.js* et codez une classe héritant de la classe *Circle*.
Dans cette nouvelle classe, nous allons étendre la classe de base. Ajouter un paramètre *dt* au constructeur.
Appelez le constructeur de la classe parente, puis stocker x et y comme position précédente du cercle.
Enfin, inversez la direction et appelez la méthode *move* de la classe parente avec ce *dt* mais négatif.
Cela permettra ainsi d'avoir une position courante et la position précédente nécessaire à l'intégration de Verlet.

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
Modifiez votre méthode move pour remettre à 0 la valeur de accelY après le mouvement.
Enfin, appliquez la gravité à chaque "tick" de votre simulation.

### Méthode *boxConstraint*
Rajoutez une méthode *boxConstraint* dans votre classe prenant une largeur et hauteur.
Lorsque le cercle touche (ou déborde) des bords de la boite, remettez la à une position valide.
Toutefois, changez la dernière position du cercle comme si le cercle avait continué à travers le "mur" (cela aura donc pour effet d'inverser la direction).
Appelez cette nouvelle méthode dans votre boucle d'animation juste après le mouvement de votre cercle pour la tester.

### Gestion des interactions
Mettez en place une interaction simple. Lors d'un click de souris, récupérez sa position.
Recherchez alors dans la collection de cercles le premier cerle en colision avec ce point.
Pour ce faire vous pouvez premièrement calculer la distance entre la position de la souris et le centre du cercle.
Ajoutez une méthode *distanceTo* dans la classe *Circle* ( et non *CircleVerlet*):

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

Enfin, appliquez une force sur le cercle en question.

### Gestion des collisions
Pour détecter les collisions entre les cercles, l'on peut comparer la distance entre les centres des deux cercles avec la somme des rayons.
Ajoutez un méthode *circleCollision* à la classe *Circle*. Cette méthode prendra un autre cercle en paramètre.
Commencez par calculer la distance entre les deux centres grâce à la méthode *distanceTo* (voir plus haut).
Si cette distance est plus grande que la somme des rayons des deux cercles, il n'y a pas de collision, donc quittez la méthode.
Sinon, calculez la distance de pénétration (overlap) et déplacez les cercles pour les séparer.
Utilisez *atan2* pour calculer l'angle de collision et déplacez les cercles en conséquence.
Voici le code JS pour calculer le déplacement à appliquer aux deux cercles:

```js
  const angle = Math.atan2(c.y - this.y, c.x - this.x);
  const overlap = this.r + c.r - distance; // où distance est la distance entre les deux centres
  const dx = Math.cos(angle) * overlap / 2;
  const dy = Math.sin(angle) * overlap / 2;
```

Soustrayez simplement *dx* et *dy* au cercle courant (this) et additionez le à l'autre.

### Ajout d'autres contraintes
Comme on l'a vu avec les rebonds sur les bords du *canvas*, l'intégration de Verlet permet de facilement ajouter des contraintes.
Nous pouvons essayez de rajouter une contrainte entre deux cercles pour les lier entre eux.
Appelons cela un *link*. Commencez par créer une classe *LinkVerlet* avec un contructeur prenant deux cercles et une distance (notre contrainte) en paramètre.
Ajoutez une méthode *update* à cette classe. De la même manière que la méthode de collision, elle calculera la distance entre les deux cercles.
L'*overlap* cette fois sera la différence entre la distance de contrainte et la distance actuelle.
Déplacez les cercles en conséquence. Dans votre programme principal, créez un lien entre deux cercles et mettez à jour ce lien à chaque *tick* de la simulation.
Pour allez plus loin, nous allons faire que certains cercles ne bougent pas. Pour ce faire, ajoutez une propriété *sticky* à la classe *CircleVerlet*.
Si cette propriété est à *true*, ne bougez pas le cercle lors de la mise à jour de la position, de même lors des collisions.
Si un cercle est *sticky*, ne le déplacez pas non plus lors de la mise à jour du lien.
Pour tester, créez un cercle *sticky* et un lien entre deux cercles (vous devriez obtenir un pendule).

Nous pouvons maintenant lier les cercles entre eux pour créer des formes plus complexes.
Par exemple une corde (ou plutôt une chaine de cercles). Pour ce faire créer une classe *Rope*.
Le constructeur prendra deux points du plan et le rayon des cercles en paramètre.
Calculez alors le nombre de cercles à créer pour relier les deux points.
Puis créez les cercles et les liens entre eux. Il faudra rendre les extrémités de la corde *sticky* pour que la corde ne tombe pas.
Ajoutez une méthode *update* à la classe *Rope* pour mettre à jour les liens entre les cercles.
Ajoutez aussi les méthodes applyForceY, applyForceX et circleCollision pour gérer les interactions.
Ces trois méthodes ne feront qu'appeler les méthodes correspondantes des cercles de la corde.
Enfin, ajoutez une méthode *draw* pour dessiner la corde.
Vous pouvez maintenant créer quelques cordes dans votre programme principal et regarder la *magie* de l'intégration de Verlet opérer.
