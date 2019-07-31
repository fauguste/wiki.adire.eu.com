---

layout: default
permalink: /git/branch/
tags: git
description: Commandes basics de gestion des branches

title: Branch
menus: git
---
# git branch


## Suppression d'une branche locale

    git branch -D BRANCH_NAME

## Suppression des toutes les branches locales n'existant plus sur le serveur

    git branch | grep -v "master" | xargs git branch -D
