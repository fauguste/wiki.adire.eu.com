---
layout: default
permalink: /git/merge/
tags: git
description: Mise à jour d'une branche à partir du master.

title: Merge
menus: git
---
## Merge

- Etape 1 : Mettre à jour la branche master en local.
````
git checkout master
git pull
````

- Etape 2 : Repasser sur la branche et appliquer le merge
````
git checkout BRANCH_NAME
git merge master
````
