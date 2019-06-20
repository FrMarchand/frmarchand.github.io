---
layout: project
type: project
image: images/qtown/qtown-square.png
title: QTown
permalink: projects/qtown
# All dates must be YYYY-MM-DD format!
date: 2015-06-06
labels:
  - C++
  - OpenGL
  - Qt
summary: Un logiciel servant à générer des environnements urbains en 3D de manière procédurale.
---

<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/qtown-header.png">

## Générateur de ville procédural

QTown est un logiciel servant à générer des environnements urbains en 3D. Il a été réalisé dans le cadre du cours de projet synthèse pour le DEC en informatique au Cégep du Vieux Montréal.

L'idée derrière ce projet était de mettre en place une procédure permettant de générer des layouts de villes très rapidement.

## Spécifications

#### Langages de programmation

* C++ 11
* GLSL

#### Outils de développement

* Visual Studio 2013
* Photoshop CS6

#### Librairies utilisées
* [Qt](http://www.qt.io/) - Cadriciel open source facilitant l'intégration de composantes d'interfaces graphiques
* [OpenGL](https://www.opengl.org/) - API open source pour le rendu graphique 2D ou 3D
* [Clipper](http://www.angusj.com/delphi/clipper.php) - Librairie open source pour l'écrêtage et le décalage de lignes et de polygones
* [Poly2tri](https://code.google.com/p/poly2tri/) - Librairie open source pour la triangulation de polygones

## Seed de génération

La première étape consiste à choisir une seed de génération. La seed est une chaîne de caractères. Elle peut être de n'importe quelle longueur et même contenir des caractères spéciaux. Si l'usager n'entre rien dans le champ désigné, une seed aléatoire de 10 caractères lui sera attribuée. La seed est ensuite hashée puis finalement transformée en entier. Cet entier est utilisé pour seeder le générateur de nombres aléatoires de la librairie standard de C++.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/seed.png" alt="seed"/>
	<figcaption>Figure 1.1: Widget permettant de saisir une seed et de démarrer la génération.</figcaption>
</figure>

## Réseau routier

Le système routier est composé d'une structure de données qui emmagasine des points 2D (x,y) représentant des segments de route. Il fait aussi appel à des fonctions génératives et à des fonctions de contraintes.

Le système utilise une structure de graphe planaire non-orienté, implémenté sous forme de liste d’adjacence. La liste contient une entrée pour chaque nœud du graphe et chacun de ces nœuds possède à son tour une liste de nœuds auxquels il est directement lié.

On initialise une liste de priorité avec une route proposée aléatoire. On initialise ensuite une liste d’adjacence vide qui servira de base pour le graphe du réseau routier. Tant que la liste de priorité n’est pas vide, on prend la route proposée avec l’indice de priorité le plus bas. On vérifie si le positionnement de la route sélectionnée respecte les contraintes locales (limites de la ville, chevauchement avec une autre rue, etc.). Si oui, le segment est ajouté à la liste d’adjacence finale. Sinon, on tente de corriger sa position. Si ce n'est pas possible, le segment est détruit.

Dans le cas ou le segment est valide, il est ensuite envoyé à une fonction de génération qui a pour but d’étendre le réseau routier en générant des branches à partir du segment envoyé en paramètre. Tous les segments générés par cette fonction sont par la suite ajoutés à la liste de priorité avec un indice de priorité égale à celui de leur ancêtre + 1000.

En suivant cet algorithme, le réseau routier peut s’étendre tant que les contraintes sont respectées. Du moment que les contraintes ne permettent plus la génération de nouveaux segments, la taille de la liste de priorité diminue pour finalement atteindre 0 et mettre fin à la boucle de génération.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/roads.png" alt="roads"/>
	<figcaption>Figure 2.1: Exemple d'un système routier généré par l'algorithme décrit ci-haut.</figcaption>
</figure>

## Préparation du graphe

Avant la prochaine étape, le graphe doit être nettoyé. La fonction de nettoyage parcourt le graphe à la recherche de segments invalides qui doivent être éliminés.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/cds-pont.png" alt="cds-pont"/>
	<figcaption>Figure 3.1: Les culs-de-sac (à gauche) et les ponts (à droite) sont des segments invalides</figcaption>
</figure>

Cette étape est nécessaire au bon fonctionnement de l'algorithme de détection des lots.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/roads-clean.png" alt="roads-cleaned"/>
	<figcaption>Figure 3.2: Le graphe original (à gauche) et le nouveau graphe nettoyé (à droite)</figcaption>
</figure>

## Génération de blocs

Une fois que le tracé des routes est complété et nettoyé, le graphe est subdivisé en blocs. Les blocs (ou lots) sont formés à partir des régions fermées du réseau routier principal. Ces régions sont déterminées par l'extraction des boucles fermées à l'intérieur du graphe. Pour y arriver, il suffit d'exécuter un algorithme de base de cycles minimaux (MCB) sur le graphe et de stocker les résultats dans une liste indépendante.

La base de cycles minimaux est définie comme l'ensemble unique des cycles minimaux à partir desquels tous les autres cycles possibles peuvent être construits.

L’algorithme commence par faire une copie du graphe et ordonne les nœuds en fonction de leur position X dans l’espace pour ensuite extraire les cycles en allant de la gauche vers la droite. Le point de départ est le nœud le plus à gauche. Quand un nœud est sélectionné, on cherche ses nœuds adjacents pour déterminer lequel sera visité. Dans le cas du premier nœud, on cherche son voisin ayant le plus petit angle dans le sens horaire. Pour tous les nœuds subséquents, on cherche le voisin ayant le plus petit angle dans le sens antihoraire.

Lorsque le chemin d’exploration retourne au nœud de départ, un cycle minimal est formé. Les nœuds qui le constituent sont alors enregistrés dans une autre structure de données.

À chaque fois qu'un cycle minimal est extrait, on recherche le cycle maximal (périmètre du graphe). Les liens entre les noeuds qui se trouvent à la fois dans le cycle minimal et maximal sont brisés, et les noeuds qui se retrouvent sans liens sont éliminés.

On répète la procédure jusqu'à ce qu'il ne reste plus de noeuds dans le graphe.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/bloc1.png" alt="bloc1"/>
	<figcaption>Figure 4.1: Exemple d'un graphe de base</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/bloc2.png" alt="bloc2"/>
	<figcaption>Figure 4.2: Recherche d'un cycle minimal</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/bloc3.png" alt="bloc3"/>
	<figcaption>Figure 4.3: Cycle minimal détecté</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/bloc4.png" alt="bloc4"/>
	<figcaption>Figure 4.4: Recherche du cycle maximal (périmètre)</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/bloc5.png" alt="bloc5"/>
	<figcaption>Figure 4.5: Détection des noeuds communs aux deux cycles</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/bloc6.png" alt="bloc6"/>
	<figcaption>Figure 4.6: Bris des liens communs aux deux cycles</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/bloc7.png" alt="bloc7"/>
	<figcaption>Figure 4.7: Un bloc a été extrait du graphe</figcaption>
</figure>

## Subdivision des blocs en bâtiments

Chaque lot généré à l’étape précédente est par la suite divisé récursivement afin de générer des empreintes pour les bâtiments. Celles qui sont trop petites ou qui n’ont pas accès à la rue sont éliminées.

Une ligne de coupure est définie perpendiculairement au côté le plus long, ce qui à pour effet de séparer le polygone en 2 à n sous-polygones (un polygone concave peut générer 3 régions et plus). Les sous-polygones reçoivent ensuite le même traitement jusqu'à ce qu'il ne soit plus possible de les partitionner (dû à des contraintes prédéfinies).

Les empreintes générées ont une liste de points qui décrivent leur périmètre ainsi qu’une valeur de hauteur générée aléatoirement.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/rec1.png" alt="rec1"/>
	<figcaption>Figure 5.1: Exemple d'une région de base</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/rec2.png" alt="rec2"/>
	<figcaption>Figure 5.2: Première ligne de coupure</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/rec3.png" alt="rec3"/>
	<figcaption>Figure 5.3: Séparation en 2 sous-régions</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/rec4.png" alt="rec4"/>
	<figcaption>Figure 5.4: Séparation récursive des sous-régions</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/rec5.png" alt="rec5"/>
	<figcaption>Figure 5.5: 2 régions ont atteint le seuil minimal</figcaption>
</figure>

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/rec6.png" alt="rec6"/>
	<figcaption>Figure 5.6: Périmètres des futur bâtiments</figcaption>
</figure>

Étant donné que les lots sont indépendants les uns des autres, l’étape de subdivision s’effectue en tandem avec l’étape de génération des lots. Dès qu’un lot est ajouté à la liste, le travail de génération de bâtiment débute immédiatement sur celui-ci.

## Création de la géométrie 3D

Une fois que l'empreinte d'un bâtiment est générée, celle-ci est triangulée à l'aide de la librairie poly2tri. À partir des points 2D du périmètre, il est possible d'effectuer une extrusion en 3 dimensions avec la valeur de hauteur associée au bâtiment. Au même moment , une section d'une des 5 textures disponibles est choisie aléatoirement pour être mappée à la géométrie. Chaque texture vient en paire. Une texture pour le jour, et une texture émissive pour la nuit.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/texture-emissive.jpg" alt="texture-emissive"/>
	<figcaption>Figure 6.1: Deux versions de la même texture. Une pour le jour (à gauche) et une pour la nuit (à droite).</figcaption>
</figure>

## Rendu 3D

Un cycle jour et nuit de base a été implémenté pour rendre le tout un peu plus agréable à l'oeil. Une source lumineuse tourne autour de la scène pour simuler l'effet du soleil sur les bâtiments. Pour donner un effet atmosphérique, chaque heure de la journée possède son propre dégradé. L'arrière-plan est une interpolation linéaire entre le dégradé de l'heure présente et celui de la prochaine heure.

Le niveau d'éclairage ambiant est calculé avec une fonction sinusoïdale qui prend en paramètre le temps de la journée. Le shader GLSL de texture utilise aussi le temps de la journée pour ajuster l'intensité des textures émissives.

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/gradients.png" alt="gradients"/>
	<figcaption>Figure 7.1: Les 24 dégradés utilisés pour simuler le temps de la journée</figcaption>
</figure>

## Résultat final

<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/screenshot1.png" alt="screenshot1"/>
	<figcaption>Figure 8.1: Vue aérienne vers midi</figcaption>
</figure>
<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/screenshot2.png" alt="screenshot2"/>
	<figcaption>Figure 8.2: Vue de soir</figcaption>
</figure>
<figure>
	<img class="ui centered image" src="{{ site.baseurl }}/images/qtown/screenshot3.png" alt="screenshot3"/>
	<figcaption>Figure 8.3: Vue le matin</figcaption>
</figure>

## Vidéo

<div class="video-wrap">
	<div class="video-container">
	<iframe src="https://www.youtube.com/embed/4-FSfN2p1Tk?rel=0" frameborder="0" allowfullscreen></iframe>
	</div>
</div>
