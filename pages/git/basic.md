---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
permalink: /git/basics.html
tags: git
description: Commandes basics d'utilisation de git

title: Basics
menus: git
---
# git
Git est un outil de gestion de configurations.

Cette article liste 2/3 commandes facile à oublier 🙂

Une documentation plus complète est disponible <a href="http://git-scm.com/book/fr/" target="_blank">ici</a>.

## Checkout

    git clone https://github.com/..../xxxxx.git

## Créer un tag

La création du tag se fait en deux étapes :

1- Création du tag en locale :

    git tag -a <TAG_NAME> -m <COMMENT>

2) Envoi du tag au serveur

    git push --tags

## Supprimer un tag

    git tag -d TAG_NAME
    git push origin :refs/tags/TAG_NAME

## Commit

1- Commit

    git commit FILE_NAME

2- Envoi du tag au serveur

    git push

## Changement de Git
Si le projet change de propriétaire, il faut

    git remote set-url origin <https://github.com/XXXX.git>

## Création d’une branche

    git checkout -b BRANCHE_NAME

Push de la branche sur le serveur :

    git push origin BRANCHE_NAME
