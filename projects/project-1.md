---
layout: project
type: project
image: images/pbr/pbr-square.png
title: PBR Material Viewer
permalink: projects/pbr
# All dates must be YYYY-MM-DD format!
date: 2019-06-10
labels:
  - C++
  - OpenGL
summary: Un visualisateur de matériaux PBR/IBL temps-réel.
---

<img class="ui centered image" src="{{ site.baseurl }}/images/pbr/pbr-header.png">

## PBR Temps-réel

Ce projet est une implémentation simplifiée de la méthode de Physically-Based Rendering/Image-Based Lighting utilisée par Epic Games et Disney telle que présentée dans la série de tutoriels disponibles sur [learnopengl.com](https://learnopengl.com/).

Le programme permet de visualiser l'effet des différentes texture maps/environment maps sur le rendu en temps-réel.

## Motivation

Le but principal de ce projet était d'approfondir mes connaissances sur divers concepts d'éclairage avancés qui n'avaient pas été abordés pendant mon cheminement universitaire. Le projet s'est concrétisé en un programme permettant d'obtenir un rendu photoréaliste en temps-réel sur un ordinateur milieu de gamme.

## Spécifications

#### Langages de programmation

* C++ 11
* GLSL

#### Outils de développement

* Visual Studio 2015

#### Librairies utilisées
* [OpenGL](https://www.opengl.org/) - API open source pour le rendu graphique 2D ou 3D
* [glad](https://glad.dav1d.de/) - Librairie de chargement d'extensions basée sur les spécifications officielles
* [glfw](https://www.glfw.org/) - Librairie pour créer des fenêtres, contextes et surfaces ainsi que recevoir les entrées et les événements
* [glm](https://glm.g-truc.net/0.9.9/index.html) - Librairie de mathématique basée sur les spécification du langage GLSL
* [dear imgui](https://github.com/ocornut/imgui) - Librairie d'interfaces utilisateur en mode immédiat
* [stb-image](https://github.com/nothings/stb) - Librairie pour charger et décoder des images à partir de fichiers ou de la mémoire
* [cmrc](https://github.com/vector-of-bool/cmrc) - Un compilateur de ressources basé sur CMake


## Fonctionnalités

Le programme permet d'effectuer les actions suivantes:

* Glisser une image HDR équirectangulaire dans la fenêtre pour automatiquement générer les cubemaps nécessaires pour le Image-Based Lighting
* Glisser des images individuelles dans les composantes du matériau (albedo, normal, metallic, roughness, ambient occlusion, displacement) pour changer l'apparence du matériau
* Ajuster l'effet de profondeur produit par la displacement map ainsi que le taille des textures
* Activer/Désactiver la rotation de l'objet
* Activer/Désactiver le mode wireframe
* Activer/Désactiver une lumière ponctuelle et la déplacer dans la scène

## Démonstration

<div class="video-wrap">
	<div class="video-container">
	<iframe src="https://www.youtube.com/embed/5Qgu7ap8vvs?rel=0" frameborder="0" allowfullscreen></iframe>
	</div>
</div>
 
## Source

<a href="https://github.com/FrMarchand/pbr-material-viewer"><i class="large github icon"></i>FrMarchand/pbr-material-viewer</a>