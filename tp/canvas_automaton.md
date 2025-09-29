.
# Canvas, animation, automates cellulaires 2D (jeu de la vie)

## Mise en place

Le premier objectif de ce TP est de continuer à travailler sur la balise _canvas_ et les animations. Le second est de s'initier aux [automates cellulaires](https://fr.wikipedia.org/wiki/Automate_cellulaire).

## Automate cellulaire 2D

Les automates cellulaires que nous allons explorer seront de type « life-like », c'est à dire ceux proche du [jeu de la vie](https://fr.wikipedia.org/wiki/Jeu_de_la_vie). L'automate sera représenté par un tableau à deux dimensions d'entiers (matrice 2D) stockant les états **vivant = 1** et **mort = 0** de chaque cellule (un tableau de booléen fonctionne aussi). Créez une classe `Automaton` (dossier `class`). Le constructeur doit recevoir au minimum la largeur et la hauteur de la matrice 2D. Ajoutez-y un paramètre optionnel pour la taille (en px) des cases et deux paramètres pour les couleurs (vivant / mort). Vous pouvez fournir des valeurs par défaut. L'initialisation de la matrice peut être vide pour le moment, une méthode d'initialisation aléatoire sera utilisée pour le remplir avec "un peu de vie".

### Méthode `randomize`

Ajoutez une méthode `randomize(prob)` qui initialise chaque cellule à l'état vivant avec la probabilité `prob` (valeur entre 0 et 1).

### Méthode `draw`

Ajoutez une méthode `draw(ctx)` qui dessine l'automate sur le _canvas_ via le contexte `ctx`. Pour positionner une cellule à l'indice `(row, col)` au bon endroit, multipliez `col` par la largeur d'une case (en px) pour obtenir la coordonnée X, et `row` par la hauteur d'une case pour obtenir la coordonnée Y. Choisissez la couleur selon l'état de la vivant/mort de la cellule et, pour simuler une grille, dessinez chaque cellule avec une marge externe d'1 px. La méthode `fillRect` est suffisante pour dessiner les cellules carrée.

Faites un test rapide en:

-   Créant un automate dont la grille prend l’entièreté du `canvas`
    
-   Initialisant avec `randomize(0.1)` pour un bruit de test.
    
-   Dessinant l'automate via `draw(ctx)`.
    

## B/S (règles Birth/Survival)

Nous utilisons la notation [Bx/Sy](https://en.wikipedia.org/wiki/Life-like_cellular_automaton#Notation_for_rules)  (ex. `B3/S23` pour le jeu de la vie) basée sur le [voisinage de Moore](https://fr.wikipedia.org/wiki/Voisinage_de_Moore). Représentez les règles de naissance et de survie par deux `Set` (ou structure équivalente) pour permettre de varier librement les automates. Il y a en tout 2^18 possibilités ! Modifiez le constructeur pour accepter ces deux ensembles de règles.

### Algorithme de transition d'une génération à la suivante

Cet algorithme décrit la manière de passer d'une génération à la suivante :

-   Pour chaque cellule, calculer le nombre de voisins vivants selon le voisinage de Moore.
    
-   Si la cellule à `(row, col)` est vivante, vérifier si son nombre de voisins est dans l'ensemble `survival` ; sinon la marquer comme devant changer d'état.
    
-   Si la cellule à `(row, col)` est morte, vérifier si son nombre de voisins est dans l'ensemble `birth` ; si oui la marquer comme devant changer d'état.
    
**Important** : ne modifiez pas immédiatement la matrice source pendant le parcours. Stockez les positions des cellules à « switcher » dans une liste (ou une structure équivalente) puis, **après** avoir parcouru toute la matrice, appliquez ces changements.        

**Gestion des bords**: Compter les voisins sur les bords de la matrice est plus complexe. Pour simplifier, essayer de transformer notre espace 2D en tore plat en utilisant le modulo euclidien (fonction a mettre dans `lib/math.js` par exemple):
```js
export function moduloEuclidian(op1, op2) {
  return ((op1 % op2) + op2) % op2;
}
```
Cette fonction rectifiera automatiquement les indices hors bornes pour simuler l'effet de bordures « enroulées » du tore.

## Animation

Dessinez votre automate dans une boucle d'animation avec [MainLoop.js](https://github.com/IceCreamYou/MainLoop.js/). Effectuez les générations dans `MainLoop.setUpdate` et effectuez le dessin dans `MainLoop.setDraw`. Vous pouvez ajuster la fréquence de génération en appelant `MainLoop.setSimulationTimestep(timestep)` où `timestep` est exprimé en millisecondes (par exemple `1000` pour une génération par seconde).

Testez avec `timestep = 1000` pour vérifier que les règles sont correctement appliquées. Vous pouvez ensuite modifier le `timestep` pour accélérer ou ralentir les générations.

## Interaction tactile / clic : changer l'état d'une cellule

Un clic (ou un touch) sur une cellule doit inverser l'état (vivant ↔ mort) de la cellule ciblée. Pour obtenir les indices `row` et `col` correspondants à la position du clic/touch  : prenez les coordonnées du pointeur relatives au document (`clientX`, `clientY`), divisez la coordonnée X par la taille du coté d'une cellule (idem pour la coordonnée Y), puis appliquez `Math.floor`. Basculez ensuite l'état de la cellule avec ces indices.

## Widget de configuration (responsive, smartphone)

**Optionel**: ajoutez un widget de configuration responsive permettant à l'utilisateur de modifier les règles **B/S** pour tester d'autres automates life-like. Offrez aussi la possibilité de régler le nombre de générations par seconde via `MainLoop.setSimulationTimestep(timestep)` et de mettre pause la simulation. Vous pouvez aussi permettre le changement de la probabilité de naissance et le reset (nouvelle génération aléatoire).

## Au-delà du Jeu de la Vie ...

Les automates cellulaires life-like ne sont qu'une famille parmi d'autres. Ils peuvent servir à la génération procédurale (génération de bruit, système de cavernes, etc), au lissage d'images, à la simulation d'écoulements, etc. Par exemple la règle [B678/S345678](https://www.jeremykun.com/2012/07/29/the-cellular-automaton-method-for-cave-generation/) fournit des structures stables intéressantes. Voila quelques exemples d'exploration : [version hexagonale](https://onivers.com/hexalife/),  [version 3D](https://onivers.com/t3d/),  [version immersive](https://onivers.com/golvr/). Vous trouverez d'autres exemples de "simulation de vie" à la fin de [la vidéo d'ego sur le jeu de la vie](https://youtu.be/eMn43As24Bo?t=1778)
